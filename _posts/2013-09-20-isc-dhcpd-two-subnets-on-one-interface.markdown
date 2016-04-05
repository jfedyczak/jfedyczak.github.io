---
layout: post
title: "ISC DHCPD two subnets on one interface"
date: 2013-09-20
tags: isc, dhcpd, two subnets
permalink: /post/isc-dhcpd-two-subnets-on-one-interface/
---
I you need to serve two or more subnets on one physical interface using `isc-dhcpd` just wrap subnet declarations in `shared-network` clause:

    shared-network interface {
        subnet 192.168.0.0 netmask 255.255.255.0 {
            option domain-name "subnet1.tld";
            option domain-name-servers 192.168.0.1;
            option routers 192.168.0.1
            option broadcast-address 192.168.0.255;
        }
        subnet 192.168.1.0 netmask 255.255.255.0 {
            option domain-name "subnet2.tld";
            option domain-name-servers 192.168.1.1;
            option routers 192.168.1.1
            option broadcast-address 192.168.1.255;
        }
    }

Replace `interface` with real interface name.