---
layout: post
author: Robin
title: 关于NSRunloop和NSTimer的理解 --- 揭开runloop的第一层面纱
tags: 开发知识 iOS
categories:
  - 开发知识
---

### 一、什么是NSRunloop

NSRunloop是Objective-C消息机制的处理模式。其作用在于有事情做的时候使用当前NSRunloop的线程工作，没有事情做的时候让当前NSRunloop的线程休眠。

NSTimer默认是添加在当前的NSRunloop中的，也可以手动添加到自己新建的NSRunloop中。

NSRunloop一直在循环检测，从线程开启到线程结束，检测Input Source（输入源）同步时间，检测Timer Source（定时源）同步事件，当检测到源的时候回执行回调处理函数，首先会产生通知，CoreFunction会向线程添加Runloop Observers（观察者）来监听事件，当事件发生的时候，处理监听到的事件。

当程序启动的时候，系统已经在主线程中加入了Runloop，保证了程序在主线程中运行起来后，就处于一种“等待”的状态，不像某些命令程序执行一次就结束了。在“等待”状态如果有接收到事件，例如定时到了或者其他线程的消息，就会执行任务，否则处于休眠状态。

Runloop是一个集合，包括监听事件源、定时源、以及通知的观察者。

**Runloop的模式：**

* default模式：几乎包括所有的输入源（除了NSConnection），NSDefaultRunLoopModel模式
* model模式：处理modal panels
* connection模式：处理NSConnection时间，属于系统内部模式，用户基本不使用
* event tracking模式：如组件拖动等，输入源UITrackingRunLoopModes模式不处理定时事件
* common modes模式：NSRunLoopCommonModes是一组可配置的通用模式，输入源与该模式关联同时输入源与该组的其他模式进行关联
  
每次运行一个Runloop，需要指定（显式或隐式）Runloop的运行模式。当相应的模式传递给Runloop时，只有与该模式对应输入源才能被检测并允许Runloop对事件进行处理，类似，也只有该模式对应的观察者才会被通知。

**例如**

1. 在Timer与Table同时执行的情况，当拖动Table的时候，Runloop会进入UITrackingRunLoopModes模式，不再会处理Timer，所以这种情况下需要将Timer加入到NSRunLoppCommonModes模式下。
2. 在滑动一个页面未松开时，此时connection不会收到消息，由于滑动时Runloop为UITrackingRunLoopModes模式，不接收输入源，此时要修改connection的mode为NSRunLoppCommonModes。

```
[scheduleInRunLoop: [NSRunLoop currentRunLoop] forMode: NSRunLoopCommonModes];
```

**runMode:beforeDate:方法**

指定Runloop模式来处理输入源，首个输入源或者Date结束后退出。
暂停当前处理的流程，转而处理其他输入源，当Date设置为`[NSDate distantFuture]`时，除非处理其他输入源的时候结束，否则永不退出处理暂停的当前流程。

```
white(A) {
  [[NSRunLoop currentRunLoop] runMode: NSDefaultRunLoopMode beforeDate: [NSDate distantFuture]];
}
```

当A为YES时，当前Runloop会一直接收处理其他输入源，当前流程不继续处理，当A为NO时，当前流程继续。

performSelector关于内存管理的执行原理是这样的执行 `[self performSelector:@selector(method1:) withObject:self.tableLayer afterDelay:3];` 的时候，系统会将tableLayer的引用计数加1，执行完这个方法时，还会将tableLayer的引用计数减1，由于延迟这时tableLayer的引用计数没有减少到0，也就导致了切换场景dealloc方法没有被调用，出现了内存泄露。

利用如下函数：

```
[NSObject cancelPreviousPerformRequestsWithTarget:self]
```


当然你也可以一个一个得这样用：

```
[NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(method1:) object:nil];
```

加上了这个以后，顺利地执行了dealloc方法。

在touchBegan里面

```
[self performSelector:@selector(longPressMethod:) withObject:nil afterDelay:longPressTime];
```

然后在end 或cancel里做判断，如果时间不够长按的时间调用：

```
[NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(longPressMethod:) object:nil]
```

取消began里的方法。


### 二、RunLoop和线程的关系
1. 主线程的Runloop默认是开启的，用户接收各种输入源。
2. 对第二线程来说，Runloop默认是没有开启的，如果需要更多的线程交互则需要手动配置和启动。如果线程执行一个长时间已经确定的任务则不需要。

### 三、RunLoop什么情况下使用
1. 使用ports或者输入源和其他线程通信的时候；
2. 在线程中使用不立即启动的Timer；
3. 使用performSeletor方法的时候，系统会启动一个线程并启动Runloop；
4. 让线程执行一个周期性的任务。

> 注：timer的创建和释放必须在同一线程中。
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];  此方法会retain timer对象的引用计数。

### 四、关于NSTimer

**1. NSTimer的准确性**

NSTimer会准确触发事件吗？答案是否定的，而且有时候会发现触发事件和想象的差距还是比较大的。NSTimer不是一个实时系统，因此不管是一次性还是周期性的timer，实际触发事件的事件可能和预想有出入。差距的大小和当前程序执行情况有关系，比如可能程序是多线程的，而timer只是添加在某一个线程Runloop的某一种执行的Runloop mode中，由于多线程通常是分时执行的，而且每次执行的mode也可能随着实际情况而发生变化。

假设添加了一个timer并制定2秒后触发某一个事件，但是恰好当前线程在执行一个连续的运算（例如大数据块的处理等），这个时候timer就会延迟到该连续运算执行结束后才会执行。重复性的timer遇到这种情况，如果延迟超过了一个周期，则会和后面的触发事件进行合并，即在一个周期内只触发一次事件。但是不管该timer的触发事件延迟有多么离谱，他后面timer的触发事件总是倍数于第一次添加的timer的间隙的。

> 原文：A repeating timer reschedules itself based on the scheduled firing time, not the actual firing time. For example, if a timer is scheduled to fire at a particular time and every 5 seconds after that, the scheduled firing time will always fall on the original 5 second time intervals, even if the actual firing time gets delayed. If the firing time is delayed so far that it passes one or more of the scheduled firing times, the timer is fired only once for that time period; the timer is then rescheduled, after firing, for the next scheduled firing time in the future.

例子：

```
- (void)applicationDidBecomeActive:(UIApplication *)application  
{  
    SvTestObject *testObject2 = [[SvTestObject alloc] init];  
    [NSTimer scheduledTimerWithTimeInterval:1 target:testObject2 selector:@selector(timerAction:) userInfo:nil repeats:YES];  
    [testObject2 release];  

    NSLog(@"Simulate busy");  
    [self performSelector:@selector(simulateBusy) withObject:nil afterDelay:3];  
}  

// 模拟当前线程正好繁忙的情况  
- (void)simulateBusy  
{  
    NSLog(@"start simulate busy!");  
    NSUInteger caculateCount = 0x0FFFFFFF;  
    CGFloat uselessValue = 0;  
    for (NSUInteger i = 0; i < caculateCount; ++i) {  
        uselessValue = i / 0.3333;  
    }  
    NSLog(@"finish simulate busy!");  
}
```

例子中首先开启了一个timer，这个timer每隔1秒调用一次target的timerAction方法，紧接着我们在3秒后调用了一个模拟线程繁忙的方法(其实就是一个大的循环)。运行程序后输出结果如下:


![NSTimer输出结果](/post_asserts/timer.jpeg)

观察结果我们可以发现，当线程空闲的时候timer的消息触发还是比较准确的，但是在36分12秒开始线程一直忙着做大量运算，知道36分14秒该运算才结束，这个时候timer才触发消息，这个线程繁忙的过程超过了一个周期，但是timer并没有连着触发两次消息，而只是触发了一次。等线程忙完以后后面的消息触发的时间仍然都是整数倍与开始我们指定的时间，这也从侧面证明，timer并不会因为触发延迟而导致后面的触发时间发生延迟。

**timer不是一种实时的机制，会存在延迟，而且延迟的程度跟当前线程的执行情况有关。**

**2. NSTimer 添加到RunLoop中才有作用**


前面的例子中我们使用的是一种便利方法，它其实是做了两件事：首先创建一个timer，然后将该timer添加到当前runloop的default mode中。也就是这个便利方法给我们造成了只要创建了timer就可以生效的错觉，我们当然可以自己创建timer，然后手动的把它添加到指定runloop的指定mode中去。

NSTimer其实也是一种资源，如果看过多线程变成指引文档的话，我们会发现所有的source如果要起作用，就得加到runloop中去。同理timer这种资源要想起作用，那肯定也需要加到runloop中才会有效。如果一个runloop里面不包含任何资源的话，运行该runloop时会立马退出。你可能会说那我们APP的主线程的runloop我们没有往其中添加任何资源，为什么它还好好的运行。我们不添加，不代表框架没有添加，如果有兴趣的话你可以打印一下main thread的runloop，你会发现有很多资源。


```
- (void)applicationDidBecomeActive:(UIApplication *)application  
{  
    [self testTimerWithOutShedule];  
}  

- (void)testTimerWithOutShedule  
{  
    NSLog(@"Test timer without shedult to runloop");  
    SvTestObject *testObject3 = [[SvTestObject alloc] init];  
    NSTimer *timer = [[NSTimer alloc] initWithFireDate:[NSDate dateWithTimeIntervalSinceNow:1] interval:1 target:testObject3 selector:@selector(timerAction:) userInfo:nil repeats:NO];  
    [testObject3 release];  
    NSLog(@"invoke release to testObject3");  
}  

- (void)applicationWillResignActive:(UIApplication *)application  
{  
    NSLog(@"SvTimerSample Will resign Avtive!");  
}
```

这个小例子中我们新建了一个timer，为它指定了有效的target和selector，并指出了1秒后触发该消息，运行结果如下:

![]({{ site.url }}/post_asserts/output.jpg)

观察发现这个消息永远也不会触发，原因很简单，我们没有将timer添加到runloop中。

**必须得把timer添加到runloop中，它才会生效。**

**3. NSTimer加到RunLoop中但不触发事件**

为什么呢？原因有两个：

1. Runloop是否运行
  每一个线程都有自己的Runloop，程序的主线程会自动是Runloop生效，但是对于自己建立的线程，它的Runloop是不会自己运行的，当需要使用它的Runloop时，就需要自己去启动。
  那么如果把一个timer添加到非主线程的Runloop中，还会按照预期的按时触发事件吗？
  
  ```
    - (void)applicationDidBecomeActive:(UIApplication *)application 
  { 
      [NSThread detachNewThreadSelector:@selector(testTimerSheduleToRunloop1) toTarget:self withObject:nil]; 
  } 
 
  // 测试把timer加到不运行的runloop上的情况 
  - (void)testTimerSheduleToRunloop1 
  { 
     NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init]; 
 
     NSLog(@"Test timer shedult to a non-running runloop"); 
     SvTestObject *testObject4 = [[SvTestObject alloc] init]; 
     NSTimer *timer = [[NSTimer alloc] initWithFireDate:[NSDate dateWithTimeIntervalSinceNow:1] interval:1 target:testObject4 selector:@selector(timerAction:) userInfo:nil repeats:NO]; 
      [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode]; 
      // 打开下面一行输出runloop的内容就可以看出，timer却是已经被添加进去 
      //NSLog(@"the thread's runloop: %@", [NSRunLoop currentRunLoop]); 
 
      // 打开下面一行, 该线程的runloop就会运行起来，timer才会起作用 
      //[[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:3]]; 
 
      [testObject4 release]; 
      NSLog(@"invoke release to testObject4"); 
 
      [pool release]; 
  } 
 
  - (void)applicationWillResignActive:(UIApplication *)application 
  { 
      NSLog(@"SvTimerSample Will resign Avtive!"); 
  }
  ```
  
上面的程序中，我们新创建了一个线程，然后创建一个timer，并把它添加当该线程的runloop当中，但是运行结果如下：

![]({{ site.url }}/post_asserts/output.jpg)

观察运行结果，我们发现这个timer知道执行退出也没有触发我们指定的方法，如果我们把上面测试程序中`//[[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:3]];`这一行的注释去掉，则timer将会正确的掉用我们指定的方法。

2. mode是否正确

我们前面自己动手添加runloop的时候，可以看到有一个参数runloopMode，这个参数是干嘛的呢？

前面提到了要想timer生效，我们就得把它添加到指定runloop的指定mode中去，通常是主线程的defalut mode。但有时我们这样做了，却仍然发现timer还是没有触发事件。这是为什么呢？

这是因为timer添加的时候，我们需要指定一个mode，因为同一线程的runloop在运行的时候，任意时刻只能处于一种mode。所以只能当程序处于这种mode的时候，timer才能得到触发事件的机会。

** 要让timer生效，必须保证该线程的runloop已启动，而且其运行的runloopmode也要匹配。**

