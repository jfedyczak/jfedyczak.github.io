---
layout: post
title: "Running express.js app behind nginx"
date: 2013-08-25
tags: express.js, nginx
permalink: /post/running-express-js-app-behind-nginx/
---
Most tutorials online suggests using reverse proxy with TCP socket. I use UNIX socket. I consider it faster - it circumvents all TCP bloat:

    fs.unlink "/tmp/app.sock", (err) ->
        app.listen "/tmp/app.sock"
        setTimeout () ->
            fs.chmod "/tmp/app.sock", 0x777, (callback) ->
        , 2000

Code above removes old socket (if any), starts express app and orders it to listen on `/tmp/app.sock` and finally changes permissions so nginx process running other user can connect to it (I believe it is not required on BSD systems).

Nginx part looks like this:

    server {
        listen <your_ip_here>:80;
        server_name example.com;
        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_pass http://unix:/tmp/app.sock;
        }
    }

That's it.

P.S. node.js part is in [CoffeeScript](http://coffeescript.org/).