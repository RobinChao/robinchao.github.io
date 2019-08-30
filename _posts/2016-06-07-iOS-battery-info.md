---
layout: post
author: Robin
title: iOS电池电量和低电量模式 --- 电池状态面面观
tags: 开发知识 iOS
categories:
  - 开发知识
---

[当电池快没电的时候，人们更愿意接受溢价。](http://www.npr.org/2016/05/17/478266839/this-is-your-brain-on-uber)，这是优步最近发布的一个数据报告中的说明。同时，优步也表明他们并没有根据利用电量的变化去设置溢价规则。

### 电池状态和电量

UIDevice 中提供了三个和电池相关的属性：


{% highlight objc  %}
@property(nonatomic,getter=isBatteryMonitoringEnabled) BOOL batteryMonitoringEnabled NS_AVAILABLE_IOS(3_0) __TVOS_PROHIBITED;  // default is NO
@property(nonatomic,readonly) UIDeviceBatteryState          batteryState NS_AVAILABLE_IOS(3_0) __TVOS_PROHIBITED;  // UIDeviceBatteryStateUnknown if monitoring disabled
@property(nonatomic,readonly) float                         batteryLevel NS_AVAILABLE_IOS(3_0) __TVOS_PROHIBITED;  // 0 .. 1.0. -1.0 if UIDeviceBatteryStateUnknown
{% endhighlight %}  

可以直接通过`batteryState`来获取电池的状态，通过`batteryLevel`获取电池的电量。

但是在获取电池的信息之前，需要手动设置`batteryMonitoringEnabled`为`YES`，否则`batteryState`将返回`UIDeviceBatteryStateUnknown`。

另外iOS系统也提供了两个通知来观察这两个属性值：

{% highlight objc  %}
UIKIT_EXTERN NSString *const UIDeviceBatteryStateDidChangeNotification;  
UIKIT_EXTERN NSString *const UIDeviceBatteryLevelDidChangeNotification;
{% endhighlight %}  


其中关于`batteryState`的值，是一个状态枚举：

{% highlight objc  %}
typedef NS_ENUM(NSInteger, UIDeviceBatteryState) {
    UIDeviceBatteryStateUnknown,
    UIDeviceBatteryStateUnplugged,   // on battery, discharging
    UIDeviceBatteryStateCharging,    // plugged in, less than 100%
    UIDeviceBatteryStateFull,        // plugged in, at 100%
} __TVOS_PROHIBITED;              // available in iPhone 3.0
{% endhighlight %}   

从其枚举值来看，可以使用电池的状态信息判断是否在充电。

当电池电量没改变`0.05`，也就是电池电量的`5%`的时候，就会触发一次`UIDeviceBatteryLevelDidChangeNotification`通知。

### 低电量模式

在iOS9中，iPhone增加了一种全新额电量模式 --- [低电量模式](https://support.apple.com/en-gb/HT205234)。在此模式下，系统会通过禁用一些特性，例如电子邮件的自动获取、后台刷新、Siri等，以达到降低能耗的目的，提升续航能力。

##### 检测电量模式	

在iOS9中，`NSProcessInfo`增加了检测低电量模式的额属性`isLowPowerModeEnabled`，开发者可以直接使用此属性，来判断当前设备是否属于低电量模式下：

{% highlight objc  %}
if ([NSProcessInfo processInfo].isLowPowerModeEnabled) {
        //LowPowerMode
    }
{% endhighlight %}  

同时，也可以注册检测低电量模式的通知，来达到低电量模式改变的响应：

{% highlight objc  %}
NSString * const NSProcessInfoPowerStateDidChangeNotification; 
{% endhighlight %}  

##### 检测低电量模式的注意点

1. 低电量模式仅仅支持iOS9以上的系统版本，需要兼容更早的版本，需要在使用前检测API的可用性；
2. 低电量模式仅仅使用在iPhone中，iPad中，`isLowPowerModeEnabled`永远返回`NO`。

##### 低电量的意义

当用户的设备处于低电量模式下的时候，开发者可根据此状态，给APP进行一些适应性改变，用来帮助用户延长电池的续航能力等。Apple给出了一下可以做的改变：

* 停止使用定位服务
* 减弱动画效果
* 停止后台任务
* 停止运动追踪 

### 参考资料

* [Energy Efficiency Guide for iOS Apps](https://developer.apple.com/library/ios/documentation/Performance/Conceptual/EnergyGuide-iOS/index.html)

* [WWDC 2015 Session 707 Achieving All-day Battery Life](https://developer.apple.com/videos/play/wwdc2015/707/)

* [Detecting low power mode](http://useyourloaf.com/blog/detecting-low-power-mode/)


