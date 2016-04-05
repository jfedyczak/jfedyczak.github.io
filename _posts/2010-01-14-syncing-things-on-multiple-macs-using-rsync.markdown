---
layout: post
title: "Syncing Things on multiple Macs using rsync"
date: 2010-01-14
tags: Things, rsync, sync
permalink: /post/syncing-things-on-multiple-macs-using-rsync/
---
I use remote SSH account to keep my [Things][1] database in sync on Macs I use. Here's the shell script:

    #!/bin/sh
    
    killall Things
    sleep 5
    rsync -zut \
        ~/Library/Application\ Support/Cultured\ Code/Things/Database.xml \
        your.host.tld:Database.xml
    rsync -zut \
        your.host.tld:Database.xml \
        ~/Library/Application\ Support/Cultured\ Code/Things/Database.xml

First, it kills Things if it's running. It is very important to synchronize database while Things cannot mess with it. Then we send local changes to the server, but only if local database is newer than remote one (`-u` option). Finally we download remote database, but again only if it is newer than local one.

iPhone sync works fine on all machines but never try to synchronize your iPhone Things with outdated version of the database!

Try to understand what you're actually doing before using this method.

[1]: http://culturedcode.com/things/