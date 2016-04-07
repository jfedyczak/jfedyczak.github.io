---
layout: post
title: "Using ffmpeg and flvtool++ to produce FLV video"
date: 2009-09-14
tags: FLV, ffmpeg, flvtool++
permalink: /post/using-ffmpeg-and-flvtool-to-produce-flv-video/
uid: 7B10CFF1-C74A-4CE2-B617-7DA8D0B62A7C
---
Here's how to convert MPEG, AVI, MOV and possibly other video formats to FLV using `ffmpeg` and add proper meta information using `flvtool++`.

First we use `ffmpeg` to generate `*.flv` file:

    ffmpeg -i 'in.mpg' \
        -b 400k -r 24 -ar 22050 -ab 96k -s 640x480 out.flv
    
This will hopefully produce 24 FPS, 400kbps video with 22050Hz, 96kbps audio. Next we have to add proper meta information using `flvtool++`:

    flvtool++ -nodump 'out.flv' 'final.flv'
    
That's it. Upload it on your webserver and use your favourite flash player to serve it to your visitors.
