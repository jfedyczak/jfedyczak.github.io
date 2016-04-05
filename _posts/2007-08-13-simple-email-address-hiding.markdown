---
layout: post
title: "Simple email address hiding"
date: 2007-08-13
tags: email hiding
permalink: /post/simple-email-address-hiding/
---
My way to hide email using JavaScript:

    --== javascript ==--
    function ml(usr,dmn) {
        document.write(
            '<a href="mailto:'+usr+'@'+dmn+'">'+usr+'@'+dmn+'</a>'
        );
    }

And how to use it in (X)HTML:

    --== html4strict ==--
    <script type="text/javascript">ml('bukaj','bukaj.net');</script>

You may also use it from PHP:

    --== php ==--
    <?php
    
        function showMail($email) {
            list($usr,$dmn)=explode('@',$email);
            echo("<script>ml('".$usr."','".$dmn."');</script>");
        }
    
    ?>
    
And how to call it from PHP:

    --== php ==--
    <?php
        showEmail("nobody@nonexistant-domain.tld");
    ?>
