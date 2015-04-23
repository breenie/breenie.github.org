---
layout: post
title: Deploying to fortrabbit from circleci
---

Deploying to Heroku from circleci is easy because it is already built in, it isn't that tricky to provide the
same functionality for fortrabbit or other even PaaS providers (probably :)).

## Generate an SSH key

Generate a new SSH key somewhere, this doesn't have to be in your `~/.ssh` directory as you won't be using it
personally.

{% highlight bash %}
$ ssh-keygen -t rsa -f /path/to/key/id_rsa
{% endhighlight %}

## Create an App-only SSH key on fortrabbit

Login to the [fortrabbit dashboard](https://dashboard.fortrabbit.com/), select the app in question and click `SSH/SFTP`.
Scroll down to `App-only SSH keys` and click `Manage App-only SSH keys`. Click the `Add new App-only SSH key` button and
enter a name for the key, usually a combination of the service name and the app for easy reference. Paste in the contents
of the public key that was generated (`id_rsa.pub`) into the `Key string` field. Click `save` to save the key.

## Register an SSH key on circleci

Login to [circleci](https://circleci.com/), find the project you're interested in and click the little cog. Scroll down
to `SSH Permissions` and click on it. Optionally enter the host name used for accessing fortrabbit via SSH (this is in
the same `SSH/SFTP` section used above). Paste in the contents of the private key that was generated (`id_rsa`). Click
`Submit` to save the key.

## Configure the circle.yml file

Open/create the `circle.yml` file in the root of the project. In this example only the `develop` branch is used, there
are many other options for which branch to use such as `master` and even feature branches.

{% highlight yaml %}
---

deployment:
    staging:
        branch: develop
        commands:
            - git push git@{GIT_URL}:{APP_NAME}.git $CIRCLE_SHA1:refs/heads/master
            - ssh {SSH_USER}@{SSH_HOST} ". /etc/profile.envvars && php htdocs/bin/console --env=prod migrations:migrate --no-interaction"
{% endhighlight %}

Replace `{APP_NAME}`, `{GIT_URL}`, `{SSH_HOST}` and `SSH_USER` with the real values from the fortrabbit dashboard.
Typically the SSH user is in the form `u-{APP_NAME}`. The last line in the example performs a database migration, this
can of course be omitted but serves as decent reference of how to execute shell commands on the fortrabbit node from
circleci.

At first I thought the node names `staging` and `production` were arbitrary but it seems they are not. The
[circleci documentation](https://circleci.com/docs/configuration#deployment) seems a little vague in that area but if
you have multiple nodes using the same branch you will get a warning on build:
`Warning: There are multiple applicable deployment sections! Using 'staging'.`. As long as the branches are unique it
_seems_ that the nodes can be named whatever. To be on the safe side stick with `staging` and `production`.

Push to the project repository on GitHub and watch the build in circleci...

{% highlight bash %}
git push git@git1.eu1.frbit.com:app.git $CIRCLE_SHA1:refs/heads/master
Warning: Permanently added 'git1.eu1.frbit.com,0.0.0.0' (RSA) to the list of known hosts.

Counting objects: 5, done.
Delta compression using up to 32 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 307 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Step1: Updating repository
remote:  -> OK
remote:
remote: Step2: Deploying (strategy: no delete, excludes: no)
remote:  -> OK
remote:
remote: Step3: Composer Hook
remote:
remote:  -> Triggering install - get a coffee
remote:    -> Up 2 date
remote:    -> Generating autoload files
remote:  -> OK
remote:
remote: > All Done <
To git@git1.eu1.frbit.com:app.git
3fe9c9f..9d431a7 da39a3ee5e6b4b0d3255bfef95601890afd80709 -> master
2342342..6786788

ssh user@xxx.eu1.frbit.com ". /etc/profile.envvars && php htdocs/bin/console --env=prod migrations:migrate --no-interaction"
Warning: Permanently added 'xxx.eu1.frbit.com,0.0.0.0' (RSA) to the list of known hosts.

  ^^                                 ^^
\(><)/ Welcome to the rabbit hole. \(><)/


Migrating up to 20150309160858 from 0

  ++ migrating 20150309140100
  ++ migrated (0.02s)

  ++ migrating 20150309160858
  ++ migrated (0.02s)

  ------------------------

  ++ finished in 0.04
  ++ 2 migrations executed
  ++ 2 sql queries

{% endhighlight; %}

Boomtimes!