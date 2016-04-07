---
layout: post
title: "Huawei E220 as SMS gateway on FreeBSD"
date: 2010-08-19
tags: huawei e220, sms gateway, freebsd
permalink: /post/huawei-e220-as-sms-gateway-on-freebsd/
uid: A2E3FF90-7AC8-4DE4-9D6C-DB8E4083972F
---
I have recently configured my home FreeBSD server as SMS gateway using Huawei E220 USB modem.

FreeBSD has built in support for this device, just add

    u3g_load="YES"
    
to your `/boot/loader.conf` and reboot or issue:

    kldload u3g
    
as root.

Here's sample Python script to read SMS data. It uses pySerial.

{% highlight python %}
# encoding: utf-8
import serial
import time
import types

# converts UCS2 to UNICODE
def ucs(t):
    w = ""
    for i in range(len(t)/4):
        w += unichr(int(t[i*4:i*4+4],16))
    return w

# sends AT command to modem and receives result
def at(k):
    global ser

    ser.write(k+"\r")
    buf = ""
    while buf[-6:] != "\r\nOK\r\n":
        buf += ser.read(1)
    buf = buf[len(k)+3:].split("\r\n")[:-2]
    return buf

# reads SMSes from modem and prints it on screen
def getSMS():
    sms = at("AT+CMGL=\"ALL\"")
    while len(sms) >= 2:
        meta = sms[0].split(",")
        idx = meta[0][6:].strip()
        sender = ucs(meta[2][1:-1])
        w = ""
        data = ucs(sms[1])
        sms = sms[2:]
        print "From: %s\nContent:\n%s\n---" % (sender,data)
        komenda("AT+CMGD="+idx)

# change your port below
ser = serial.Serial("/dev/ttyU0.0",115200,timeout=5,rtscts=True)    
ser.open()
at("AT")
# at("AT+CPIN=****") # enter PIN if you have one
at("AT+CMGF=1") # puts modem in text mode
at("AT+CPMS=\"SM\"") # change SMS storage
at("AT+CSCS=\"UCS2\"") # change encoding
while True:
    getSMS()
    time.sleep(0.5)
{% endhighlight %}
