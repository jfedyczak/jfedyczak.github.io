---
layout: post
title: "Simple email address hiding"
date: 2007-08-13
tags: email hiding
permalink: /post/simple-email-address-hiding/
uid: CD93B21C-1A4E-4D4C-B52E-0452CFF6F1B1
---
My way to hide email using JavaScript:

{% highlight js %}
function ml(usr, dmn) {
    document.write(
        '<a href="mailto:'+usr+'@'+dmn+'">'+usr+'@'+dmn+'</a>'
    );
}
{% endhighlight %}

And how to use it in (X)HTML:

{% highlight html %}
<script type="text/javascript">ml('bukaj','bukaj.net');</script>
{% endhighlight %}

You may also use it from PHP:

{% highlight php %}
<?php

    function showMail($email) {
        list($usr,$dmn)=explode('@',$email);
        echo("<script>ml('".$usr."','".$dmn."');</script>");
    }

?>
{% endhighlight %}
    
And how to call it from PHP:

{% highlight php %}
<?php
    showEmail("nobody@nonexistant-domain.tld");
?>
{% endhighlight %}
