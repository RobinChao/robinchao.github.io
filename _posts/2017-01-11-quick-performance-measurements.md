---
layout: post
author: Robin
title: 如何衡量代码执行的耗时 --- mach_absolute_time的使用
tags: 开发知识 iOS
categories:
  - 开发知识
--- 

 
在代码的编写过程中，通常会对比较耗时的任务进行耗时测量，并不断的优化代码的逻辑以提高代码的执行效率，减小执行耗时。在iOS系统中，有很多和时间相关的方法，这里主要介绍一个快速和简单的方法。

在iOS的底层，提供了一个 **mach_absolute_time** 函数，该函数获取到的是CPU的tickcount的计数值，并可以通过**mach_timebase_info**函数获取到纳秒级的精确度。
 

{% highlight objc %} 
#import <mach/mach_time.h> // for mach_absolute_time

double MachTimeToSecs(uint64_t time)
{
    mach_timebase_info_data_t timebase;
    mach_timebase_info(&timebase);
    return (double)time * (double)timebase.numer / 
                (double)timebase.denom / 1e9;
}
    
- (void)profileDoSomething
{  
    uint64_t begin = mach_absolute_time();
    [self doSomething];
    uint64_t end = mach_absolute_time();
    NSLog(@"Time taken to doSomething %g s", 
                MachTimeToSecs(end - begin));
}
{% endhighlight %}
 
上述方法可以获取到一段代码或者一个函数的执行耗时，但是CPU线程之间的调度肯定要花费时间，所以只能尽可能的精确。
