---
layout: post
title: "Express.js internals crash course"
date: 2013-08-26
tags: express.js
permalink: /post/express-js-internals-crash-course/
---
## Introduction

This is short story of my understanding of [express.js](http://expressjs.com) web application stack. All source code below is in [CoffeeScript](http://coffeescript.org), which I find more readable and easier to write than vanilla JavaScript.

## Minimal express.js app

Express.js is actually very thin (yet powerful) layer over [node's http stuff](http://nodejs.org/api/http.html). Let's start with minimal example:

{% highlight coffee %}
express = require 'express'

app = express()

app.configure ->
    app.use express.bodyParser()
    app.use express.cookieParser()
    app.use app.router
    app.use express.static "#{__dirname}/public"

app.get '/', (req, res) ->
    res.send '<body>Hello</body>'

app.listen 3000
{% endhighlight %}

Here's what happens when `GET` request comes:

1. `bodyParser` middleware is called - it decodes different kinds of http request body.
2.  `cookieParser` is called - it decodes `Cookie` headers and populates `req.cookies`.
3. `app.router` is called. It traverses `app.routes` array to find matching route using regexps. If it finds one, route is being executed and all ends here. If it doesn't express proceeds to next middleware.
4. Next middleware - `static` is called. It checks for matching static files in `/public` directory. When found - file is being sent back to client in response.

## Routes internal object

Here's `app.routes` content for above example (via `console.log app.routes`):

    { get:
        [ { path: '/',
            method: 'get',
            callbacks: [Object],
            keys: [],
            regexp: /^\/\/?$/i } ] }

`app.routes` object was automatically populated by `app.get` call.

## Custom application middleware

Every middleware takes three arguments:

{% highlight coffee %}
customMiddleware = (req, res, next) ->
    console.log "Hello from middleware! User IP: #{req.ip}"
    next()
{% endhighlight %}

`req` and `res` are request and response objects and `next` is a callback which is set by `app.use` to next middleware in request processing chain. Try putting following line after `cookieParser`:

{% highlight coffee %}
app.use customMiddleware
{% endhighlight %}

Running application and requesting `http://127.0.0.1:3000` will yield following line on the console for every request:

    Hello from middleware! User IP: 127.0.0.1

It is important to call `next` inside middleware. In other case request processing chain will stop on your middleware (what can be desired in some cases).
