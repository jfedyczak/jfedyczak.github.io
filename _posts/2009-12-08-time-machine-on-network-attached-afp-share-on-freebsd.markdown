---
layout: post
title: "Time Machine on network attached AFP share on FreeBSD"
date: 2009-12-08
tags: Time Machine, netatalk, FreeBSD, OS X
permalink: /post/time-machine-on-network-attached-afp-share-on-freebsd/
uid: 0E048941-0ABB-4409-B82F-DCB13AFFFBD6
---
#Intoduction#
**OS X Lion (10.7) Note:** Lion requires some features not available in `netatalk` since version 2.2.

Although you can easily find several tutorials on that subject, I compiled it into one that worked for me. Here you can find instructions necessary to create OS X Time Machine backup on remote FreeBSD machine (I haven't checked but I'm pretty sure that similar procedure works on Linux also). Description presented here is NOT meant to be implemented literally. Try to treat it as a guideline and adopt it to your own needs and environment.

#Netatalk installation#

I used ports to install netatalk (AFP implementation). As `root`:

    cd /usr/ports/net/netatalk/ && make install distclean

Be sure to enable Zeroconf (Bonjour) during configuration.
    
to `/etc/rc.conf` add:

    netatalk_enable="YES"
    afpd_enable="YES"
    cnid_metad_enable="YES"

Put following content in `/usr/local/etc/afpd.conf`:

    "Time Machine" -uamlist uams_dhx2.so

Put following appropriate line in `/usr/local/etc/AppleVolumes.default` (change path and allowed user as needed):

    /home/jf/tm/    "Time Machine" allow:jf options:tm,usedots
    
Start netatalk:

    /usr/local/etc/rc.d/netatalk start

#Create image#

Since filesystem provided by FreeBSD is not quite compatible with Time Machine, you have to create your own file system image. Use `Disk Utility` to create image:

* Save As: `MachineName_MacAddress`
`MachineName` as read from 'System Preferences / Sharing' Computer Name field. MacAddress has to be ethernet address of en0 - as displayed in `ifconfig en0` result in ehternet line - without colons. For example: `jf_010203040506`. Always use ethernet interface. Wireless interface won't work.
* Name: `Time Machine`
* Size: Custom - greater than your HD size
* Format: I use `Mac OS Extended (Journaled)` but I'm not sure which one is the best.
* Partitions: `Single partition - Apple partition map`
* Image format: `sparse bundle disk image`

Resulting file should have `.sparsebundle` extension. Copy it to your remote FreeBSD volume using Finder. Remember to "Connect As" correct user. Now you have to allow unsupported volumes to be seen by Time Machine; in terminal issue:

    defaults write com.apple.systempreferences TMShowUnsupportedNetworkVolumes 1
    
Now you are ready to setup Time Machine in System Preferences. You should see remote volume as "Time Machine".

Have fun.
