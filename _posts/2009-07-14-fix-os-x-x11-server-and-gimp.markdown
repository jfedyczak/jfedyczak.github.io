---
layout: post
title: "Fix OS X's X11 Server and  Gimp"
date: 2009-07-14
tags: gimp, os x, x11
permalink: /post/fix-os-x-x11-server-and-gimp/
uid: D50C4F78-17EA-4BB4-9CF2-0B52C0E47603
---
The way Gimp looks and feels on OS X has been driving me crazy, so I've forced myself and found appropriate settings:

    defaults write org.x.X11 wm_click_through -bool true
    defaults write org.x.X11 dpi -int 75

This sanitizes window focusing and sets resolution to the value at which font sizes in Gimp seem normal.

Next I've changed in `Edit / Preferences / Window Management` tab:

 * `Hint for the toolbox` -> `Keep above`
 * `Hint for other docks` -> `Keep above`
 * `Activate the focused image` -> checked
 * `Save window positions on exit` -> checked

That's it.
