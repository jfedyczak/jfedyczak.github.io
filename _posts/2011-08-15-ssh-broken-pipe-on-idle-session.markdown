---
layout: post
title: "SSH Broken pipe on idle session"
date: 2011-08-15
tags: SSH, broken pipe, idle, timeout
permalink: /post/ssh-broken-pipe-on-idle-session/
uid: D8240AF3-01B2-46A2-9995-67537A5A6E37
---
After upgrading to OS X Lion, I've experienced broken pipe in SSH connections after some idle time. To prevent that, I've added

    ServerAliveInterval 30
To `/etc/ssh_config`. Hope it helps someone too.
