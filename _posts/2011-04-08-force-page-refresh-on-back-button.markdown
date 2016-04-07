---
layout: post
title: "Force page refresh on back button"
date: 2011-04-08
tags: refresh, back button
permalink: /post/force-page-refresh-on-back-button/
uid: A6F44D98-07D9-4D2A-A9B0-6B25D4E799E7
---
I've tried to force a page to be downloaded again by browser when user clicks back button, but nothing worked. I appears that modern browsers have separate cache for pages, which stores complete state of a page (including JavaScript generated DOM elements), so when users presses back button, previous page is shown instantly in state the user has left it. If you want to force browser to reload page on back button, add `onunload=""` to your (X)HTML body element:

{% highlight html %}
<body onunload="">
{% endhighlight %}

This disables special cache and forces page reload when user presses back button. Think twice before you use it. Fact of needing such solution is a hint your site navigation concept is flawed.
