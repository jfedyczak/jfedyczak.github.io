---
layout: post
title: "Snow Leopard clean install + restore from Time Machine"
date: 2009-09-04
tags: snow leopard, os x, time machine, install
permalink: /post/snow-leopard-clean-install-restore-from-time-machine/
uid: 6A979563-4FDE-481A-97D2-035A6C4E3226
---
I've just installed Snow Leopard (OS X 10.6) on my Unibody 13.3" MB. Everything works perfectly fine. What I did step by step:

1. Updated Time Machine backup (on external USB 2.0 HDD).
2. Booted from Snow Leopard retail ($29) installation disk (I had to hold "C" key during boot to force boot from DVD)
3. Ran Disk Utility (from Utilities menu) and erased my Macintosh HD partition.
4. Ran standard installation on clean partition.
5. Created user `test`.
1. Installed X11 and XCode from installation DVD.
6. Installed iLife from my Leopard 10.5 installation DVD (bundled with my MB).
7. Updated everything to recent version (I'm still stuck with iLife 2008).
1. Installed software that I use (mainly VMware Fusion, Textmate, Transmit, NeoOffice and macports stuff).
9. Ran Migration Assistant and restored my user and system settings (without applications) from my Time Machine backup.
10. Removed user `test`.

Everything works 2x faster. VMWare Fusion works with my 64-bit guest systems. iPhone syncs ok. So I'm up and running in about 3 hours. I don't know is it Snow Leopard or system reinstall, but it was worth it :)
