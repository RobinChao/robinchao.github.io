---
layout: post
title: DNS 解析实践 --- iOS下如何优雅的获取域名的DNS
date: 2017-03-20 09:46:59 +0800
categories: 开发知识
tags: DNS network iOS
---


![](/assets/getaddrinfo.png)


1. 引入必要头文件

``` c
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
```

2. 核心代码

``` c
int lookup_host (const char *host)
{
    struct addrinfo hints, *res;
    int errcode;
    char addrstr[100];
    void *ptr;
    
    memset (&hints, 0, sizeof (hints));
    hints.ai_family = PF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags |= AI_CANONNAME;
    
    errcode = getaddrinfo (host, NULL, &hints, &res);
    if (errcode) {
        if (errcode == EAI_SYSTEM)
            fprintf(stderr, "looking up www.example.com: %s\n", strerror(errno));
        else
            fprintf(stderr, "looking up www.example.com: %s\n", gai_strerror(errcode));
        
        return -1;
    }
    
    printf ("Host: %s\n", host);
    while (res)
    {
        inet_ntop (res->ai_family, res->ai_addr->sa_data, addrstr, 100);
        
        switch (res->ai_family)
        {
            case AF_INET:
                ptr = &((struct sockaddr_in *) res->ai_addr)->sin_addr;
                break;
            case AF_INET6:
                ptr = &((struct sockaddr_in6 *) res->ai_addr)->sin6_addr;
                break;
        }
        inet_ntop (res->ai_family, ptr, addrstr, 100);
        printf ("IPv%d address: %s (%s)\n", res->ai_family == PF_INET6 ? 6 : 4,
                addrstr, res->ai_canonname);
        res = res->ai_next;
    }
    
    return 0;
}
```

这里使用了**getaddrinfo**，是Apple在推出IPv6之后建议使用的API，兼容了IPv4和IPv6。网路上大多数方式都还在使用**gethostbyname**，这个方法已经不建议使用了，详见Apple文档。


### ChangeHostForTrust

``` objc

static inline SecTrustRef ChangeHostForTrust(SecTrustRef trust, NSString * trustHostname)
{
    if ( ! trustHostname || [trustHostname isEqualToString:@""]) {
        return trust;
    }
    
    CFMutableArrayRef newTrustPolicies = CFArrayCreateMutable(kCFAllocatorDefault, 0, &kCFTypeArrayCallBacks);
    
    SecPolicyRef sslPolicy = SecPolicyCreateSSL(true, (CFStringRef)trustHostname);
    
    CFArrayAppendValue(newTrustPolicies, sslPolicy);
    
    /* This technique works in iOS 2 and later, or
     OS X v10.7 and later */
    
    CFMutableArrayRef certificates = CFArrayCreateMutable(kCFAllocatorDefault, 0, &kCFTypeArrayCallBacks);
    
    /* Copy the certificates from the original trust object */
    CFIndex count = SecTrustGetCertificateCount(trust);
    CFIndex i=0;
    for (i = 0; i < count; i++) {
        SecCertificateRef item = SecTrustGetCertificateAtIndex(trust, i);
        CFArrayAppendValue(certificates, item);
    }
    
    /* Create a new trust object */
    SecTrustRef newtrust = NULL;
    if (SecTrustCreateWithCertificates(certificates, newTrustPolicies, &newtrust) != errSecSuccess) {
        /* Probably a good spot to log something. */
        
        return NULL;
    }
    
    return newtrust;
}
```
