---
layout: post
title: 苹果邮件事件 --- 三八妇女节的躁动
date: 2017-03-08 17:35:08 +0800
categories: 小记录
tags: 小记录，Apple email
---


![](/assets/Appstore.jpg)

### 不平凡的妇女节

公元2017年3月8日，一个注定不普通的日子，对于广大的妇女同志们，半天放假、礼物、蛋糕、祝福、红包等等都属于她们。但是对于地球上东方国家---中国来说，有一批人在这一天注定不得安宁，这就是庞大的iOS开发工程师们。

这一天清晨，各种社交媒体上都涌现了一封平常但不平凡的邮件，AppStore的审核团队发出的警告邮件，内容如下：

> Dear Developer,

> Your app, extension, and/or linked framework appears to contain code designed explicitly with the capability to change your app’s behavior or functionality after App Review approval, which is not in compliance with section 3.3.2 of the Apple Developer Program License Agreement and App Store Review Guideline 2.5.2. This code, combined with a remote resource, can facilitate significant changes to your app’s behavior compared to when it was initially reviewed for the App Store. While you may not be using this functionality currently, it has the potential to load private frameworks, private methods, and enable future feature changes.

> This includes any code which passes arbitrary parameters to dynamic methods such as dlopen(), dlsym(), respondsToSelector:, performSelector:, method_exchangeImplementations(), and running remote scripts in order to change app behavior or call SPI, based on the contents of the downloaded script. Even if the remote resource is not intentionally malicious, it could easily be hijacked via a Man In The Middle (MiTM) attack, which can pose a serious security vulnerability to users of your app.

> Please perform an in-depth review of your app and remove any code, frameworks, or SDKs that fall in line with the functionality described above before submitting the next update for your app for review.

> Best regards,

> App Store Review

### 简单YY

这样一封邮件为何导致庞大的中国iOS开发者躁动不安，Github上issures铺天盖地，相关网站访问速度下降，AppStore访问速度下降等等呢？且听慢慢道来。一切的源头都是源自AppStore审核规则中的一条：

> 2.5.2 Apps should be self-contained in their bundles, and may not read or write data outside the designated container area, nor may they download, install, or execute code, including other iOS, watchOS, macOS, or tvOS apps.

这条规则是在2016年WWDC之后更新的，明确的指出所有被执行的代码都应该包含在当前App内，不能下载代码到App中进行执行，无论是JS代码还是Objective-C代码或者其他形式的代码，都违反了这条规则。但是2016年更新这条规则之后，苹果一直没有严格的“执行”，或许是由于苹果的审核机制还不完善吧，但是今天，苹果开始严格执行这条规则，开始清理AppStore上的应用，向“疑似”违反这条鬼策的App发警告邮件，并拒绝违反这条规则App上架。

在邮件中，苹果提到了几个方法，这些方法本身没有问题，但是就是因为这些方法，让苹果的安全机制收到了威胁。

> dlopen(), dlsym(), respondsToSelector:, performSelector:, method_exchangeImplementations()

这些方法几乎都是可以和iOS底层API打交道的。但是详读邮件内容，其实苹果并不是完全禁止使用这些API，只是在使用这些API的时候，不能使用外部引入的参数。

例如这样写就不会有问题：

``` objc
if([self.delegate respondsToSelector: @selector(myDelegateMethod)]) {
   [self.delegate performSelector: @selector(myDelegateMethod)];
}
```
但是这样写可能就会被拒绝：

``` objc
NSString *remotelyLoadedString = .... (download from your backend)
[self performSelector: NSSelectorFromString(remotelyLoadedString)];
```

另外苹果还可能会扫描特定的类名或者字段，例如`JSPatch`、`Rollout`、`HotFix`等等类似的。至于为什么会扫描类似的这些类名或者字段，还望自行脑补。


### 对未来的猜想

首先，苹果是绝对不会同意任何的应用绕过他们的审核而下发代码的，无论是修复bug，还是更新功能。但是仔细想想，如果AppStore由于此类问题导致安全问题，国内的国情可能就是cut AppStore了，庞大的中国用户苹果怎么会放弃，也不可能放弃。

另一方面，此种下发代码的方式，的确会有很多的安全问题，道高一尺魔高一丈，下发代码的方式总是会有漏洞，导致安全问题、隐私问题，这些恐怕是苹果永远不会接受的。

因此，不管是iOS开发还是那些PM们，尽快忘掉这个技术吧，在iOS上目前可能还有办法规避，但是苹果帝国不会那么容易就让你”薅羊毛“的。


**那么问题来了？到底这些技术能导致什么问题呢？下篇文章我会尝试进行解读，望持续关注，谢谢！**
