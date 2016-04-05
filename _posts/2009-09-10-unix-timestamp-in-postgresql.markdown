---
layout: post
title: "Unix timestamp in PostgreSQL"
date: 2009-09-10
tags: unix timestamp, postgresql, convert, postgres, epoch
permalink: /post/unix-timestamp-in-postgresql/
---
Since I'm looking for it every time, I've decided to put it here.

To convert UNIX Timestamp to Postgres timestamp:

    SELECT TIMESTAMP WITH TIME ZONE 'epoch' +
        INTERVAL '1 second' * 1252606560 AS postgres_timestamp;

To convert Postgres timestamp to UNIX timestamp:

    SELECT extract('epoch' FROM now())::int AS unix_timestamp;

Skip the `::int` cast to get fractional part.