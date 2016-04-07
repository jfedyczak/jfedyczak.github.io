---
layout: post
title: "PHP: Distinguish unset array element from null array element"
date: 2009-07-15
tags: php, isset, is_null
permalink: /post/php-distinguish-unset-array-element-from-null-array-element/
uid: F6C37398-203F-409A-B898-76C9D125C1A8
---
Let's execute following code:

{% highlight php %}
<?php
    $a = array();
    $a[0] = 1;
    $a[1] = null;
    $a[3] = 1;

    for ($i = 0; $i < 4; $i++) {
        echo("index ".$i.":\n");
        echo(" isset: ");
        var_dump(isset($a[$i]));
        echo(" is_null: ");
        var_dump(is_null($a[$i]));
        echo("\n");
    }
?>
{% endhighlight %}
The result:

    index 0:
     isset: bool(true)
     is_null: bool(false)

    index 1:
     isset: bool(false)
     is_null: bool(true)

    index 2:
     isset: bool(false)
     is_null: bool(true)

    index 3:
     isset: bool(true)
     is_null: bool(false)
Notice, that for `$a[2]` as well as for `$a[1]` `is_null` and `isset` return true, so you can't distinguish between set variable with null value from completely unset variable. However you can use `array_key_exists` to do so:

{% highlight php %}
<?php
    $a = array();
    $a[0] = 1;
    $a[1] = null;
    $a[3] = 1;

    for ($i = 0; $i < 4; $i++) {
        echo("index ".$i.":\n");
        echo(" isset: ");
        var_dump(isset($a[$i]));
        echo(" is_null: ");
        var_dump(is_null($a[$i]));
        echo(" array_key_exists: ");
        var_dump(array_key_exists($i,$a));
        echo("\n");
    }
?>
{% endhighlight %}

This results with:

    index 0:
     isset: bool(true)
     is_null: bool(false)
     array_key_exists: bool(true)

    index 1:
     isset: bool(false)
     is_null: bool(true)
     array_key_exists: bool(true)

    index 2:
     isset: bool(false)
     is_null: bool(true)
     array_key_exists: bool(false)

    index 3:
     isset: bool(true)
     is_null: bool(false)
     array_key_exists: bool(true)

Hope this will help you. It took me a while to figure out.
