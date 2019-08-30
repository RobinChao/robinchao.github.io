---
layout: post
author: Robin
title: Captive Web Protal 认证原理 --- CNA技术的基本原理
tags: 学习调研
categories:
  - 学习调研
--- 

### 一、表现形式

当在部分机场、宾馆、餐馆等场所时，wifi列表中会有无需密码的wifi可供使用，当点击连接后，设备会自动弹出或者用户在访问站点时被强制转到一个登陆网页，需要手机号等其他信息以进行认证。

### 二、工作原理

此技术的实现称做`Captive Network Assistant（CNA）`，几乎所有的可联网的设备均支持，而其认证方式称之为`Captive Portal`，而以网页的方式弹出认证页面的方式又称为`Captive Web Protal（CWP）`。

1. 基本的工作流程描述

	a. 在指定的场所范围内，网络提供商利用无线AP，广播无密码的SSID信号（wifi）；

	b. 需要联网的设备加入到其中一个wifi网络（wifi列表中选择某一个wifi）；

	c. 需要联网的设备主动申请DNS解析一个预支持的网站IP，以确认网络是否畅通。如果畅通，调到步骤 `h`;

	d. 网络设备返回Captive portal地址的IP；

	e. 用户的设备主动打开Captive portal网站，进行登录认证，登录失败则会踢出当前网络；

	f. 认证成功后，网络设备会记录下当前设备的MAC地址和所分配的IP，并开通网络访问；

	g. 用户设备在网页发生跳转后，网络设备会在此检测DNS，当反馈的IP正常后，提示用户可以正常上网（对于iOS，认证网页右上角有出现Done）；
	
	h. 网络设备在提供服务的过程中，可以限制网络开通情况（例如：使用时长、流量限制等）。
	
### 三、相关设备的支持

##### iOS 
iOS设备（包括Mac系列）在系统服务中有一个`Captive Network Assistant`的服务，其中iOS系统中内置了一个app：`/System/Library/CoreServices/Captive Network Assistant.app`，专门用来进行`CWP`的操作。

##### Android 

在Android系统中内置了一个`WifiWatchDog`服务，功能和iOS的类似。源码可见：[Link](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/4.0.1_r1/android/net/wifi/WifiWatchdogStateMachine.java#WifiWatchdogStateMachine.isWalledGardenConnection%28%29)


##### Windows

具体请见：[Link](http://blog.superuser.com/2011/05/16/windows-7-network-awareness/) 
 
### 四、iOS Captive Network Assistant info

* [Use captive Wi-Fi networks on your iPhone, iPad, or iPod touch](https://support.apple.com/en-us/HT204497)

* [如何关闭Captive Network Assistant（CNA）自动弹出页面](https://discussionschinese.apple.com/thread/44410?start=0&tstart=0)

* [How to automatically login to captive portals on OS X?](http://apple.stackexchange.com/questions/45418/how-to-automatically-login-to-captive-portals-on-os-x)

* [How to create WiFi popup login page](http://stackoverflow.com/questions/3615147/how-to-create-wifi-popup-login-page)



