---
layout: post
title: "Using bluetooth for measuring city traffic conditions"
date: 2009-05-06
tags: bluetooth, traffic, tracking
permalink: /post/using-bluetooth-for-measuring-city-traffic-conditions/
uid: 8E1FA01F-1161-4697-A444-2180FF1BA607
---
I came up with this idea while I was pairing my mobile phone with my Mac. The idea is very simple. I've tested this "in the field" and it seems to work pretty well. Instead of tracking cars, you can track mobile phones of drivers and passengers. Many people leave their bluetooth devices in discoverable mode. This allows us to track them by periodically sending inquiry messages. This way it is possible to obtain MAC addresses of tracked devices. MAC addresses of BT devices are theoretically unique. By placing "sensors" near road, you can collect MAC addresses along with time of people (cars) passing by. When collected MAC address appear somewhere else - you can easily calculate time needed to travel from first location to another one. By placing several "sensors" in critical locations in a city, you can easily obtain information about current (and past) status of traffic conditions. Probability is your friend here. You don't have to collect every single MAC address. Simple average or median filters out any anomalies. Below simple proof of concept BT scanners for OS X:

{% highlight objc %}
#import <Foundation/NSObject.h>
#import <IOBluetooth/objc/IOBluetoothDevice.h>
#import <IOBluetooth/objc/IOBluetoothDeviceInquiry.h>
#import <stdio.h>

@interface Discoverer: NSObject {}
-(void) deviceInquiryComplete: (IOBluetoothDeviceInquiry*) sender 
                        error: (IOReturn) error
                      aborted: (BOOL) aborted;
-(void) deviceInquiryDeviceFound: (IOBluetoothDeviceInquiry*) sender
                          device: (IOBluetoothDevice*) device;
@end
@implementation Discoverer
-(void) deviceInquiryComplete: (IOBluetoothDeviceInquiry*) sender 
                        error: (IOReturn) error
                      aborted: (BOOL) aborted
{
//    printf("inquiry complete\n");
    [sender clearFoundDevices];
    [sender start];
}
-(void) deviceInquiryDeviceFound: (IOBluetoothDeviceInquiry*) sender
                          device: (IOBluetoothDevice*) device
{
    printf("%s\n", [[device getAddressString] UTF8String]);
    fflush(stdout);
}
@end

int main( int argc, const char *argv[] ) 
{
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];

    Discoverer *d = [[Discoverer alloc] init];

    IOBluetoothDeviceInquiry *bdi = 
    [[IOBluetoothDeviceInquiry alloc] init];
    [bdi setDelegate: d];

    [bdi setInquiryLength: 3];
    [bdi setUpdateNewDeviceNames:false];
    [bdi start];

    CFRunLoopRun();

    [bdi release];
    [d release];
    [pool release];
    return 0;
}
{% endhighlight %}

and for Linux (BlueZ stack):

{% highlight c %}
#include <stdio.h>
#include <stdio.h>
#include <errno.h>
#include <ctype.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <getopt.h>
#include <sys/param.h>
#include <sys/ioctl.h>
#include <sys/socket.h>

#include <bluetooth/bluetooth.h>
#include <bluetooth/hci.h>
#include <bluetooth/hci_lib.h>

static void spinq(int dev_id)
{
    uint8_t lap[3] = { 0x33, 0x8b, 0x9e };
    struct hci_request rq;
    periodic_inquiry_cp cp;
    int opt, dd;

    if (dev_id < 0)
        dev_id = hci_get_route(NULL);

    dd = hci_open_dev(dev_id);
    if (dd < 0) {
        perror("Device open failed");
        exit(EXIT_FAILURE);
    }

    memset(&cp, 0, sizeof(cp));
    memcpy(cp.lap, lap, 3);
    cp.max_period = htobs(16);
    cp.min_period = htobs(10);
    cp.length     = 1;
    cp.num_rsp    = 0;

    memset(&rq, 0, sizeof(rq));
    rq.ogf    = OGF_LINK_CTL;
    rq.ocf    = OCF_PERIODIC_INQUIRY;
    rq.cparam = &cp;
    rq.clen   = PERIODIC_INQUIRY_CP_SIZE;

    if (hci_send_req(dd, &rq, 100) < 0) {
        perror("Periodic inquiry failed");
        exit(EXIT_FAILURE);
    }

    hci_close_dev(dd);
}


int main() {
//    spinq(-1);

    inquiry_info *info = NULL;
    uint8_t lap[3] = { 0x33, 0x8b, 0x9e };
    int num_rsp, length, flags;
    char addr[18];
    int i, l, opt;
    int dev_id;

    while (1) {
        dev_id = -1;
        length  = 3;    /* ~10 seconds */
        num_rsp = 0;
        flags   = 0;

        flags |= IREQ_CACHE_FLUSH;

        num_rsp = hci_inquiry(dev_id, length, num_rsp, lap, &info, flags);
        if (num_rsp < 0) {
            perror("Inquiry failed.");
            exit(1);
        }

        for (i = 0; i < num_rsp; i++) {
            ba2str(&(info+i)->bdaddr, addr);
            printf("%s\n",
                addr, btohs((info+i)->clock_offset)
            );
            fflush(stdout);
        }

    }
    bt_free(info);

}
{% endhighlight %}

This is all in alpha / proof of concept stage. Use it at will.
