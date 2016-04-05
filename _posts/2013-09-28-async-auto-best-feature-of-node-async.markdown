---
layout: post
title: "Async.auto - best feature of node async"
date: 2013-09-28
tags: async, node.js, auto, callback hell
permalink: /post/async-auto-best-feature-of-node-async/
---
## Introduction

If you think that callback hell is a real problem, you obviously missed [async](https://github.com/caolan/async) module. I know most popular method are probably `async.series` and `async.parallel` but my personal favourite is `async.auto`. I consider it the ultimate solution to callback hell problem and much more...

## Callback hell?

Let's try do this:

 1. write domain name to a file
 - check if the file exists
 - read domain name from the file
 - check IP address, MX record and reverse address for the domain
 - delete the file
 - display results

Here's a typical way do acomplish above:

    dns = require 'dns'
    fs = require 'fs'

    fs.writeFile 'domain.txt', 'google.com', (e) ->
        fs.exists 'domain.txt', (exists) ->
            if exists
                fs.readFile 'domain.txt', 'utf8', (e, domain) ->
                    dns.lookup domain, 4, (e, addr, family) ->
                        dns.resolveMx domain, (e, addresses) ->
                            dns.reverse addr, (e, domains) ->
                                fs.unlink 'domain.txt', (e) ->
                                    console.log "#{domain}:"
                                    console.log " -- ip:  #{addr}"
                                    console.log " -- mx:  #{addresses[0].exchange}"
                                    console.log " -- rev: #{domains}"
            else
                console.log "file doesn't exist"
Result:

    google.com:
     -- ip:  173.194.70.139
     -- mx:  alt1.aspmx.l.google.com
     -- rev: fa-in-f139.1e100.net

You should also see resulting JavaScript code...

## async.auto to the rescue!

Now we'll do this again but using `async.auto`:

    dns = require 'dns'
    fs = require 'fs'
    async = require 'async' # install with npm install async

    # takes object with tasks and requirement
    async.auto
        # 'write' task has no requirements
        write: (cb) ->
            fs.writeFile 'domain.txt', 'google.com', (e) ->
                cb e
        # requires 'write' task result
        exists: ['write', (cb) ->
            fs.exists 'domain.txt', (exists) ->
                return cb 'file doesn't exist' unless exists
                cb null
        ]
        # requires 'exists' task result
        domain: ['exists', (cb) ->
            fs.readFile 'domain.txt', 'utf8', cb
        ]
        # requires 'domain' task result - passed in res argument
        ip: ['domain', (cb, res) ->
            dns.lookup res.domain, 4, (e, addr, family) ->
                cb null, addr
        ]
        # also requires 'domain' task result
        mx: ['domain', (cb, res) ->
            dns.resolveMx res.domain, (e, addresses) ->
                cb null, addresses[0].exchange
        ]
        # requires 'ip' task result
        rev: ['ip', (cb, res) ->
            dns.reverse res.ip, (e, domains) ->
                cb null, domains
        ]
        # requires 'domain' task
        unlink: ['domain', (cb) ->
            fs.unlink 'domain.txt', cb        
        ]
    # final callback - requires all tasks to finish
    , (e, res) ->
        # if any task return error:
        return console.log e if e
        console.log "#{res.domain}:"
        console.log " -- ip:  #{res.ip}"
        console.log " -- mx:  #{res.mx}"
        console.log " -- rev: #{res.rev}"

As you can see `async.auto` takes list of tasks with requirements. Every task is provided with results from required tasks list. If any task pass error argument to callback function, processing stops and final callback is being called with error argument. When all tasks are successfully finished, final callback is called with object containings results.

But that's not all. There's a bonus! For example `ip` and `mx` tasks are being executed in parallel, because they don't depend on each other's result! If you track dependencies carefully, you'll see that actual processing scenario is as follows:

<img src="/assets/async.svg" style="width: 100%" alt="Flow diagram" />

As you can see, the code is faster, because everything that can is being executed in parallel. You can also easily change order of execution by modifying dependencies. This should be in core!
