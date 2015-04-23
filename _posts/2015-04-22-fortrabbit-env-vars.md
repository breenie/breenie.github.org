---
layout: post
title: Loading environment variables via SSH on fortrabbit
---

The environment variables on fortrabbit reside in `/etc/profile.envvars`. This file must be executed before any app
level environment variables will be loads.

Issuing the command:

`ssh user@xxx.eu1.frbit.com php htdocs/bin/console --env=prod migrations:status --no-interaction`

Yields the following:

`[PDOException]`<br />
`SQLSTATE[HY000] [2002] No such file or directory`

Appending `. /etc/profile.envvars &&` to the command means the command executes correctly.

`ssh user@xxx.eu1.frbit.com ". /etc/profile.envvars && htdocs/bin/console --env=prod migrations:status --no-interaction"`

Premature hair loss abated.