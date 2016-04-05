---
layout: post
title: "Why using exceptions in node.js is a bad idea"
date: 2013-09-29
tags: node.js, exceptions
permalink: /post/why-using-exceptions-in-node-js-is-a-bad-idea/
---
## Introduction

Using exceptions in [node.js](http://nodejs.org/) is pointless because of its aynchronous nature. I'll try to show you why.

## Destined to fail

Here's simple dot server code. Everything what may brake is in `try/catch` clause:

{% highlight coffee %}
dotServer = ->
    console.log "."
    setTimeout dotServer, 1000

dotServer()
{% endhighlight %}

try
    console.log "zero seconds"
    setTimeout ->
        console.log "four seconds"
        throw error
    , 4000
    console.log "zero seconds second time"
    throw error
    console.log "never happens"
catch error
    console.log "exception!"

console.log "out of try/catch clause"

Results:

    .
    zero seconds
    zero seconds second time
    exception!
    out of try/catch clause
    .
    .
    .
    four seconds

    /Users/bukaj/Dropbox/blog/exc.coffee:15
          throw error;
                ^
    undefined

As you can see once we're out of `try/catch` clause, throwing exception causes process to exit. So much for our dot server.

## When to use exception

Exceptions are perfectly legitimate in synchronous parts of code. For example:

{% highlight coffee %}
try
    console.log "processing JSON string"
    res = JSON.parse "{wtf"
    console.log "got it!"
catch SyntaxError
    console.log "wrong syntax!"
{% endhighlight %}

This results in:

    processing JSON string
    wrong syntax!
