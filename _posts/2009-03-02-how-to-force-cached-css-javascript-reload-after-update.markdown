---
layout: post
title: "How to force cached CSS/JavaScript reload after update"
date: 2009-03-02
tags: CSS, js, reload, cache, php
permalink: /post/how-to-force-cached-css-javascript-reload-after-update/
---
Everytime I update my JS scripts or CSS style sheets, browser cache prevents current version from being downloaded and uses old files instead. To avoid it, you have to inform browser, that file has been modified. Simplest way of doing it is to change script name. You can do this manually but it may become pretty tedious - especially with multiple places in your code to maintain it. The easy way to automate it using PHP is to add a parameter to script / style section:
    
{% highlight php %}
...
<link href="style.css?time=<?php 
    echo(filemtime("style.css")); 
?>" rel="stylesheet" type="text/css" />
...
{% endhighlight %}
    
This makes style sheet being loaded from server everytime it is changed, but keeps it cached otherwise (`filemtime` should change only when file actually changes). You can use this on script files also.
