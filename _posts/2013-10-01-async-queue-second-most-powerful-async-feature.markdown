---
layout: post
title: "Async.queue - second most powerful async feature"
date: 2013-10-01
tags: node.js, async queue, crawling
permalink: /post/async-queue-second-most-powerful-async-feature/
---
## Introduction

Recently I wrote about [crawling a site](/post/website-crawling-with-node-js) using [cheerio](https://github.com/MatthewMueller/cheerio) and [request](https://github.com/mikeal/request) modules, but in this post I'd like to show you my second favourite feature of [async](https://github.com/caolan/async) which is `queue`. It simplifies concurrent processing of repeatable tasks.

## request meets async.queue

`async.queue` allow creating of task queue with preset concurrency:

{% highlight coffeescript %}
crawlingQueue = async.queue crawlUrl, 3
{% endhighlight %}

This creates `crawlingQueue` with worker function `crawlUrl` and concurrency of 3, which means that no more than 3 workers process tasks in the same time.

To add new tasks to the queue, use the `.push` method:

{% highlight coffeescript %}
crawlingQueue.push siteToCrawl
{% endhighlight %}

You can also pass optional second argument - a callback - called when processing of particular task finishes.

First argument could also be an array allowing you to add multiple tasks in one call:

{% highlight coffeescript %}
crawlingQueue.push urls
{% endhighlight %}

To execute some code after all tasks have been processed, use `.drain` callback:

{% highlight coffeescript %}
crawlingQueue.drain = ->
    console.log " -- internal URLS: #{visitedUrls.length}"
    console.log " -- external URLS: #{externalUrls.length}"
    console.log "    --> #{url}" for url in externalUrls
{% endhighlight %}

Below there's a complete example. It allows crawling all internal pages of a site and finding all external URLs leading outside crawled domain. It uses 3 crawling workers. Do not use it for serious tasks.

{% highlight coffeescript %}
async = require 'async'
request = require 'request'
cheerio = require 'cheerio'
URL = require 'url'

siteToCrawl = 'http://jakub.fedyczak.net/'
restrictDomain = URL.parse(siteToCrawl).hostname

visitedUrls = []
externalUrls = []

crawlUrl = (url, callback) ->
    return callback null if url in visitedUrls
    visitedUrls.push url
    request
        uri: url
        method: 'GET'
        timeout: 5000
    , (err, res, body) ->
        return callback err if err
        $ = cheerio.load body
        urls = []
        $('a').each () ->
            href = $(@).attr 'href'
            return true unless href
            href = URL.resolve url, href
            hostname = URL.parse(href).hostname
            if hostname is restrictDomain
                urls.push href unless (href in urls) or (href in visitedUrls)
            else
                externalUrls.push href unless href in externalUrls
        console.log " -- #{url} (#{urls.length})"
        crawlingQueue.push urls
        callback null

crawlingQueue = async.queue crawlUrl, 3
crawlingQueue.drain = ->
    console.log " -- internal URLS: #{visitedUrls.length}"
    console.log " -- external URLS: #{externalUrls.length}"
    console.log "    --> #{url}" for url in externalUrls
crawlingQueue.push siteToCrawl
{% endhighlight %}

And here's example result:

     -- http://jakub.fedyczak.net/ (32)
     -- http://jakub.fedyczak.net/post/async-auto-best-feature-of-node-async/ (29)
     -- http://jakub.fedyczak.net/post/why-using-exceptions-in-node-js-is-a-bad-idea/ (28)
     -- http://jakub.fedyczak.net/post/website-crawling-with-node-js/ (27)
     -- http://jakub.fedyczak.net/post/rocksmith-real-tone-usb-cable/ (26)
     -- http://jakub.fedyczak.net/post/isc-dhcpd-two-subnets-on-one-interface/ (25)
     -- http://jakub.fedyczak.net/post/freebsd-vlan-configuration/ (24)
     -- http://jakub.fedyczak.net/post/merge-upsert-example-in-postgresql-using-cte/ (23)
     -- http://jakub.fedyczak.net/post/express-js-internals-crash-course/ (22)
     -- http://jakub.fedyczak.net/post/running-express-js-app-behind-nginx/ (21)
     -- http://jakub.fedyczak.net/post/sparrow-mail-migration-to-another-mac/ (20)
     -- http://jakub.fedyczak.net/post/postgresql-with-recursive-tree-traversing-example/ (19)
     -- http://jakub.fedyczak.net/post/ssh-broken-pipe-on-idle-session/ (18)
     -- http://jakub.fedyczak.net/post/huawei-e173-on-ubuntu/ (17)
     -- http://jakub.fedyczak.net/post/iphone-4-maximum-video-length-and-size/ (16)
     -- http://jakub.fedyczak.net/post/force-page-refresh-on-back-button/ (15)
     -- http://jakub.fedyczak.net/post/huawei-e220-as-sms-gateway-on-freebsd/ (14)
     -- http://jakub.fedyczak.net/post/ios-4-on-iphone-3g/ (13)
     -- http://jakub.fedyczak.net/post/syncing-things-on-multiple-macs-using-rsync/ (12)
     -- http://jakub.fedyczak.net/post/time-machine-on-network-attached-afp-share-on-freebsd/ (11)
     -- http://jakub.fedyczak.net/post/slow-count-in-postgresql/ (10)
     -- http://jakub.fedyczak.net/post/using-ffmpeg-and-flvtool-to-produce-flv-video/ (9)
     -- http://jakub.fedyczak.net/post/unix-timestamp-in-postgresql/ (8)
     -- http://jakub.fedyczak.net/post/snow-leopard-clean-install-restore-from-time-machine/ (7)
     -- http://jakub.fedyczak.net/post/php-distinguish-unset-array-element-from-null-array-element/ (6)
     -- http://jakub.fedyczak.net/post/fix-os-x-x11-server-and-gimp/ (5)
     -- http://jakub.fedyczak.net/post/how-to-force-cached-css-javascript-reload-after-update/ (4)
     -- http://jakub.fedyczak.net/post/using-bluetooth-for-measuring-city-traffic-conditions/ (3)
     -- http://jakub.fedyczak.net/post/international-domain-names-idn-decoder/ (2)
     -- http://jakub.fedyczak.net/post/regexps-in-check-constraints/ (1)
     -- http://jakub.fedyczak.net/post/postgresql-insert-returning-update-returning/ (0)
     -- http://jakub.fedyczak.net/post/pgbouncer-and-server-reset-query/ (0)
     -- http://jakub.fedyczak.net/post/simple-email-address-hiding/ (0)
     -- internal URLS: 33
     -- external URLS: 17
        --> http://serpstat.pl/
        --> http://mimeo.it/
        --> https://plus.google.com/110570871381903458999?rel=author
        --> https://github.com/caolan/async
        --> http://nodejs.org/
        --> https://github.com/mikeal/request
        --> https://github.com/MatthewMueller/cheerio
        --> http://coffeescript.org/
        --> http://www.postgresql.org/docs/9.2/interactive/queries-with.html
        --> http://expressjs.com/
        --> http://nodejs.org/api/http.html
        --> http://sparrowapp.com/
        --> http://culturedcode.com/things/
        --> http://en.wikipedia.org/wiki/Multiversion_concurrency_control
        --> http://www.ietf.org/rfc/rfc3493.txt
        --> http://en.wikipedia.org/wiki/Internationalized_domain_name
        --> http://en.wikipedia.org/wiki/UTF-8

As you can see, async provides simple yet very powerful soltions to asynchronous processing.
