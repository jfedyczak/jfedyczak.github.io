---
layout: post
title: "Unix timestamp in PostgreSQL"
date: 2009-09-10
tags: unix timestamp, postgresql, convert, postgres, epoch
permalink: /post/unix-timestamp-in-postgresql/
uid: 3B2F9F2F-73C0-4971-B9BE-764111E561CC
---
Since I'm looking for it every time, I've decided to put it here.

To convert UNIX Timestamp to Postgres timestamp:

{% highlight sql %}
SELECT TIMESTAMP WITH TIME ZONE 'epoch' +
    INTERVAL '1 second' * 1252606560 AS postgres_timestamp;
{% endhighlight %}

To convert Postgres timestamp to UNIX timestamp:

{% highlight sql %}
SELECT extract('epoch' FROM now())::int AS unix_timestamp;
{% endhighlight %}

Skip the `::int` cast to get fractional part.
