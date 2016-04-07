---
layout: post
title: "International Domain Names (IDN) decoder"
date: 2009-02-26
tags: IDN, decoder, International Domain Names, punycode, PHP IDN decoder
permalink: /post/international-domain-names-idn-decoder/
uid: 1C7AE182-5A96-4FEB-9C83-40C0CA2F260B
---
IDN stands for internationalized domain name. It's a domain containing non-ASCII international character(s). As it is not directly supported by DNS servers translating names to IP addresses, punycode has been used to encode international (Unicode) characters to ASCII characters making it backwards compatible with existing software. This is strictly specified in RFC 3492. For example domain name containing polish characters:

    zażółć.pl

can be encoded as:

    xn--za-6ja4f8n1l.pl

In such form it can be used as any other ordinary ASCII domain. You can register such domeain, setup your DNS for it etc. Now let's say you'd like to use dynamically created URL IDN addresses. For example city-name.example.com. You've setup apache `ServerAlias *.example.com` directive in Apache config and you're ready to receive requests with your `index.php`, which might look like this:

{% highlight php %}
<?php
    header("Content-Type: text/html; charset=UTF-8");

    $hostname = $_SERVER['HTTP_HOST'];
    
    echo("You're at ".$hostname);
?>
{% endhighlight %}
    
Now opening `warsaw.example.com` will work as expected. But opening for example `łódź.example.com` will leave you with raw punycode encoded ASCII string. To make it work, try using decoding class below:

{% highlight php %}
<?php

class IDN {

    // adapt bias for punycode algorithm
    private static function punyAdapt(
        $delta,
        $numpoints,
        $firsttime
    ) {
        $delta = $firsttime ? $delta / 700 : $delta / 2; 
        $delta += $delta / $numpoints;
        for ($k = 0; $delta > 455; $k += 36)
            $delta = intval($delta / 35);
        return $k + (36 * $delta) / ($delta + 38);
    }

    // translate character to punycode number
    private static function decodeDigit($cp) {
        $cp = strtolower($cp);
        if ($cp >= 'a' && $cp <= 'z')
            return ord($cp) - ord('a');
        elseif ($cp >= '0' && $cp <= '9')
            return ord($cp) - ord('0')+26;
    }

    // make utf8 string from unicode codepoint number
    private static function utf8($cp) {
        if ($cp < 128) return chr($cp);
        if ($cp < 2048) 
            return chr(192+($cp >> 6)).chr(128+($cp & 63));
        if ($cp < 65536) return 
            chr(224+($cp >> 12)).
            chr(128+(($cp >> 6) & 63)).
            chr(128+($cp & 63));
        if ($cp < 2097152) return 
            chr(240+($cp >> 18)).
            chr(128+(($cp >> 12) & 63)).
            chr(128+(($cp >> 6) & 63)).
            chr(128+($cp & 63));
        // it should never get here 
    }

    // main decoding function
    private static function decodePart($input) {
        if (substr($input,0,4) != "xn--") // prefix check...
            return $input;
        $input = substr($input,4); // discard prefix
        $a = explode("-",$input);
        if (count($a) > 1) {
            $input = str_split(array_pop($a));
            $output = str_split(implode("-",$a));
        } else {
            $output = array();
            $input = str_split($input);
        }
        $n = 128; $i = 0; $bias = 72; // init punycode vars
        while (!empty($input)) {
            $oldi = $i;
            $w = 1;
            for ($k = 36;;$k += 36) {
                $digit = IDN::decodeDigit(array_shift($input));
                $i += $digit * $w;
                if ($k <= $bias) $t = 1;
                elseif ($k >= $bias + 26) $t = 26;
                else $t = $k - $bias;
                if ($digit < $t) break;
                $w *= intval(36 - $t);
            }
            $bias = IDN::punyAdapt(
                $i-$oldi,
                count($output)+1,
                $oldi == 0
            );
            $n += intval($i / (count($output) + 1));
            $i %= count($output) + 1;
            array_splice($output,$i,0,array(IDN::utf8($n)));
            $i++;
        }
        return implode("",$output);
    }
    
    public static function decodeIDN($name) {
        // split it, parse it and put it back together
        return 
            implode(
                ".",
                array_map("IDN::decodePart",explode(".",$name))
            );
    }

}

header("Content-Type: text/html; charset=UTF-8");

$hostname = IDN::decodeIDN($_SERVER['HTTP_HOST']);

echo("You're at ".$hostname);

?>
{% endhighlight %}
    
I know it's not perfect. I tried to keep it short and use least PHP extensions as possible. I just hope I gave you general idea behind IDN and dealing with it in PHP.
    
Further reading:

* [RFC 3492](http://www.ietf.org/rfc/rfc3493.txt)
* [IDN](http://en.wikipedia.org/wiki/Internationalized_domain_name)
* [UTF-8](http://en.wikipedia.org/wiki/UTF-8)
