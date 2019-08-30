---
layout: post
author: Robin
title: 如何获取当前应用的CompilationSDK --- infoDictionary数据解析
tags: 开发知识 iOS
categories:
  - 开发知识
--- 

在分析应用的ipa文件时，发现ipa中的`info.plist`文件和Xcode中的不太一样，于是对比后发现，ipa中的info.plist中包含了很多有用的信息，比如打包应用时的`BuildMachineOSBuild`、`CFBundleIdentifier`、`CFBundleShortVersionString`、`DTCompiler`、`DTSDKName`、`DTXcode`、`DTXcodeBuild`等等信息。

这里的信息其实都是可以直接获取的，在iOS SDK中的提供了直接获取此文件的方法`infoDictionary`，通过此方法可以直接获取到当前文件内容转换后的字典，通过此字典既可以获取到这些数据信息。

* SDK Version

{% highlight objc  %}
+ (NSString *)buildVersion {
    NSMutableCharacterSet *characterSet = [[NSCharacterSet
                                            decimalDigitCharacterSet] mutableCopy];
    [characterSet formUnionWithCharacterSet:[NSCharacterSet punctuationCharacterSet]];
    
    // get only those things in characterSet from the SDK name
    NSString *SDKName = [[NSBundle mainBundle] infoDictionary][@"DTSDKName"];
    NSArray *components = [[SDKName componentsSeparatedByCharactersInSet:
                            [characterSet invertedSet]]
                           filteredArrayUsingPredicate:
                           [NSPredicate predicateWithFormat:@"length != 0"]];
    
    if([components count]) return components[0];
    return nil;
}
{% endhighlight %}  

* Xcode Version

{% highlight objc  %}
+ (NSString *)XcodeVersion {
    NSMutableCharacterSet *characterSet = [[NSCharacterSet
                                            decimalDigitCharacterSet] mutableCopy];
    [characterSet formUnionWithCharacterSet:[NSCharacterSet punctuationCharacterSet]];
    
    // get only those things in characterSet from the SDK name
    NSString *SDKName = [[NSBundle mainBundle] infoDictionary][@"DTXcode"];
    NSArray *components = [[SDKName componentsSeparatedByCharactersInSet:
                            [characterSet invertedSet]]
                           filteredArrayUsingPredicate:
                           [NSPredicate predicateWithFormat:@"length != 0"]];
    
    if([components count]) return components[0];
    return nil;
}
{% endhighlight %}

其他信息的获取方式大同小异，其中也有很多value为Array的数据，在获取的时候需要注意！

info.plist样例：

![]({{ site.url }}/assets/info1.png)
![]({{ site.url }}/assets/info2.png) 
