---
layout: post
title: "FreeBSD VLAN configuration"
date: 2013-09-20
tags: freebsd, vlan
permalink: /post/freebsd-vlan-configuration/
---
## Introduction

This is short text about setting 802.11q VLANs in FreeBSD. It uses new `/etc/rc.conf` syntax.

## /etc/rc.conf changes

Let's assume that there are two physical interfaces `bce0` and `bce1`. First thing - rename them for better readability:

    ifconfig_bce0_name="outside"
    ifconfig_bce1_name="inside"

Next - setup `outside` interface - one without VLANs:

    ifconfig_outside="10.0.0.2/30"

Now the VLAN part:

    ifconfig_inside="up"

This is sometimes required if there is no IP address assigned to untagged interface.

VLANs declarations for `inside` interface:

    vlans_inside="switches users rest"

Above declaration defines three VLANs. Now we have to assign VLAN numbers to them:

    create_args_switches="vlan 1"
    create_args_users="vlan 99"
    create_args_rest="vlan 98"

After that, we can configre each VLAN using standard way:

    ifconfig_switches="10.235.0.1/24"
    ifconfig_users="192.168.0.1/24"
    ifconfig_rest="192.168.1.0/24"

That's it.