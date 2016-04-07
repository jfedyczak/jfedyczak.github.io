---
layout: post
title: "Huawei E173 on Ubuntu"
date: 2011-04-16
tags: huawei e173, ubuntu
permalink: /post/huawei-e173-on-ubuntu/
uid: 0199D0CA-7A51-478E-84FC-6E6BE7FC516E
---
I've been trying to get my Huawei E173 modem working on Ubuntu. I'm no Ubuntu expert and it took me half a day to get it working, so here's what I eventually did:

Use HyperTerminal on Windows to switch device into modem mode (factory default is to make it a CD-ROM and a microSD card reader). Install all the drivers and connect to correct COM port. In my case it was `COM3`, but it was just my case. I've been trying to issued `AT` on all COM ports and finally I saw `OK`. Then I issued `AT^U2DIAG=0` which also resulted in `OK`. If you don't want to do this, you can try using `usb_modeswitch`.

On Ubuntu I've inserted the device, and issued `lsusb`:

    Bus 001 Device 003: ID 12d1:1c05 Huawei Technologies Co., Ltd.
    
I've edited `/etc/modules` and added a line:
    
    usbserial vendor=0x12d1 product=0x1c05
    
Reboot.

Configure your mobile broadband connection using gnome network manager.

Use at your own risk (especially the Windows part).
    
