---
layout: post
author: Robin
title: iOS平台数据流量的获取 --- 如何获取不同网络环境下的数据流量？
tags: 开发知识 iOS
categories:
  - 开发知识
  - 网络
--- 
 
{% highlight objc  %}
	#import <ifaddrs.h>
	#import <sys/socket.h>
	#import <net/if.h>
{% endhighlight %} 


{% highlight objc  %}
	+(int)get3GFlowIOBytes{
    struct ifaddrs *ifa_list= 0, *ifa;
    if (getifaddrs(&ifa_list)== -1) {
        return 0;
    }
    uint32_t iBytes = 0;
    uint32_t oBytes = 0;
    for (ifa = ifa_list; ifa; ifa = ifa->ifa_next) {
        if (AF_LINK != ifa->ifa_addr->sa_family)
            continue;
        if (!(ifa->ifa_flags& IFF_UP) &&!(ifa->ifa_flags & IFF_RUNNING))
            continue;
        if (ifa->ifa_data == 0)
            continue;
        if (!strcmp(ifa->ifa_name,"pdp_ip0")) {
            struct if_data *if_data = (struct if_data*)ifa->ifa_data;
            iBytes += if_data->ifi_ibytes;
            oBytes += if_data->ifi_obytes;
            NSLog(@"%s :iBytes is %d, oBytes is %d",ifa->ifa_name, iBytes, oBytes);
        }
    }
    freeifaddrs(ifa_list);
    return iBytes + oBytes;
}


+(long)getWifiInterfaceBytes{
    struct ifaddrs *ifa_list = 0, *ifa;
    if (getifaddrs(&ifa_list) == -1) {
        return 0;
    }
    uint32_t iBytes = 0;
    uint32_t oBytes = 0;
    for (ifa = ifa_list; ifa; ifa = ifa->ifa_next) {
        if (AF_LINK != ifa->ifa_addr->sa_family)
            continue;
        if (!(ifa->ifa_flags & IFF_UP) && !(ifa->ifa_flags & IFF_RUNNING))
            continue;
        if (ifa->ifa_data == 0)
            continue;
        if (strncmp(ifa->ifa_name, "lo", 2)) {
            struct if_data *if_data = (struct if_data *)ifa->ifa_data;
            iBytes += if_data->ifi_ibytes;
            oBytes += if_data->ifi_obytes;
            NSLog(@"%s :iBytes is %d, oBytes is %d", ifa->ifa_name, iBytes, oBytes);
        }
    }
    freeifaddrs(ifa_list);
    return iBytes+oBytes;
}
{% endhighlight %} 