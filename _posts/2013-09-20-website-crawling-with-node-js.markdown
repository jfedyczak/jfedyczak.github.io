---
layout: post
title: "Website crawling with node.js"
date: 2013-09-20
tags: node, request, cheerio
permalink: /post/website-crawling-with-node-js/
uid: 400EB18C-AC2E-4172-9628-0D2411097896
---
##Introduction

When you need to quickly scrape some data from a website, here's a simple solution using:

 - [node.js](http://nodejs.org)
 - [request](https://github.com/mikeal/request) - HTTP client
 - [cheerio](https://github.com/MatthewMueller/cheerio) - HTML parser with jQuery-like API

I'll ignore error checking for simplicity. Don't ever do that - especially with node.js - it'll kick your ass. As always - everything using [CoffeeScript](http://coffeescript.org).

##Crawling a site

We need some initialization:

{% highlight coffee %}
request = require 'request'
cheerio = require 'cheerio'
async = require 'async'
{% endhighlight %}

Initialize cookie jar in case we're handling cookie-aware site using HTTP redirects:

{% highlight coffee %}
jar = request.jar()
{% endhighlight %}

Now start actual crawling:

{% highlight coffee %}
request
    followRedirect: true
    uri: 'http://cnn.com/'
    timeout: 10000
    jar: jar
, (err, res, body) ->
    $ = cheerio.load body
    $('a').each ->
        console.log "#{$(@).text()} -> #{$(@).attr 'href'}"
{% endhighlight %}

We use cheerio to parse request body and find anchor text of each link on site. Results:

    CNN Shop -> http://www.turnerstoreonline.com/
    Site map -> /sitemap/
    CNN Partner Hotels -> http://partners.cnn.com/
    CNN en ESPAÑOL -> /espanol/
    CNN Chile -> http://www.cnnchile.com
    CNN México -> http://www.cnnmexico.com
    العربية -> http://arabic.cnn.com/
    日本語 -> http://www.cnn.co.jp/
    Türkçe -> http://www.cnnturk.com/
    Turner Broadcasting System, Inc. -> http://www.turner.com/
    Terms of service -> /interactive_legal.html
    Privacy guidelines -> /privacy.html
    Ad choices -> /services/ad.choices/
    Advertise with us -> http://www.cnnmediainfo.com/
    License our content -> /intlsyndication/
    About us -> /about/
    Contact us -> /feedback/
    Work for us -> http://www.turner.com/careers/
    Help -> /help/
    CNN TV -> /CNNI/
    HLN -> /HLN/
    Transcripts -> http://transcripts.cnn.com/TRANSCRIPTS/
