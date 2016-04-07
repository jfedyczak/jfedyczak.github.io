---
layout: post
title: "Upgrade PostgreSQL on OS X using homebrew"
date: 2013-10-14
tags: postgresql, upgrade, homebrew
permalink: /post/upgrade-postgresql-on-os-x-using-homebrew/
uid: 0E633FC8-A6BE-4158-B703-DB582DC07831
---
These are short instructions about upgrading [PostgreSQL](http://postgresql.org) using [Homebrew](http://brew.sh) - great OS X package manager.

## Upgrading instructions

I'm assuming you have PostgreSQL installed using `brew` and running under `launchctl`.

Start with data backup (may take a while):

    /usr/local/bin/pg_dumpall > pgbackup.sql

Stop old PostgreSQL version:

    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist

Update Homebrew database:

    brew update

Upgrade PostgreSQL:

    brew upgrade postgresql

Remove old cluster (warning: this will remove all your data and PostgreSQL settings):

    rm -rf /usr/local/var/postgres/*

Initialize new cluster (I use polish locale and UTF8, but YMMV):

    /usr/local/bin/initdb /usr/local/var/postgres --locale pl_PL.UTF8 -E utf8

Start new version:

    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist

Import old data to new cluster:

    /usr/local/bin/psql -d postgres -f pgbackup.sql

You can remove old backup after veryfing that everything works.

I've tested above procedure on several machines while upgrading from 9.2 to 9.3. Use on your own risk.
