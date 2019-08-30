---
layout: post
author: Robin
title: 在 Swift 中自定义计时器 --- 再也不怕计时器不好使用了
tags: 开发知识 iOS
categories:
  - 开发知识
  - Swift
--- 

> 简单的才是王道。这个世上的各种新科技，都是在把复杂的事物简单化。
  
  
NSTimer类是iOS应用程序的一个主要的类，但是它有潜藏着各种复杂的难易预料的问题。例如，当一个计时器到
期，它对目标对象选择器的回调无法再纯粹的Swift类、struct中使用，这迫使使用NSObject-derived类。同
时，运行时保持着计时器对象的强引用，这可能导致意想不到的问题。在同一个线程中，当计时器被注册后，为了打破强引用，必须停止计时器（调用`invalidate()`）。但是当停止一次之后，计时器将无法再次使用。


不仅要问：“在Swift 3 中是否还会保留NSTimer类的呢？”  为了能够适应不断的变化，尝试自己来设计一个工具类，实现类型NSTimer中的一些常用的API。针对API的设想：  
  
* 尽量简单、干净的类
* 完全的 Swift 语法
* 以闭包为基础
* 允许保持对象的弱引用
* 线程安全的订阅和取消timers

首先，来设想一下希望如何使用API，写一些假象的代码段：  
  
1. 在某段时间后执行block中的代码，且仅执行一次：   
 
```  
  Repeat.once(after: 1) {
  	print("running once after 1 second.")
  }
```  
  
2. 隔某段时间执行block中的代码，且重复执行：  
  
```  
  Repeat.every(seconds: 5) {
  	print("another 5 seconds has gone by...")
  }
```  
  
3. 针对一个timer，还需要停止，因此API还需要有一个返回值：  
  
```  
  let id = Repeat.once(after: 2) {
  	print("this is never going to run")
  }
  id.invalidate()
```  
  
4. 在重复执行时候，还需要根据某些条件，执行不同的方式，例如停止、继续：  
  
```  
  Repeat.after(seconds: 5) {
  	print("still here...")
  	if shouldStop {
  		return .Stop
  	} else {
  		return .Repeat
  	}
  }
```  
  
5. 在单次执行后，嵌套执行重复定时任务：  
  
```  
  Repeat.once(after: 3) {
  	var count = 3
  	// Start off refreshing every 1 second
  	Repeat.every(seconds: 1) {
  		print("count = \(count)")
  		count += 1

  		// if get to 10, stop
		if count = 10 {
			return .Stop
		} else {
		// at 5, go faster
			return .RepeatAfter(0.5)
		} else {
		// otherwist repeat at the current rate
			return .Repeat
		}		 
  	}
  }
```  
    
  
到此，设想的使用都已经完成了。回顾一下最初的设想：  
  
1. 尽量简单、干净的类，其中有一些成熟的变量供我们使用
2. 能够保持timer闭包中的代码如期的执行
3. 使用闭包使得Swift代码更加的简洁。

下面定义整体的API函数方法名：  
  
```  
  // Execute closure once after given delay (in seconds)
  Repeat.once(after: NSTimerInterval, closure: () -> ()) -> RepeatSubscriberId
  
  // Execute closure idefinitely with given delay
  Repeat.every(seconds: NSTimerInterval, closure: () -> ()) -> RepeatSubscriberId
  
  // Execute closure after given delay. Further executions/delays controlled by 
  // return value from closure, which can be .Stop, .Repeat  or .RepeatAfter(NSTimeInterval)
  Repeat.after(seconds: NSTimeInterval, closure: () -> Repeat.Result) -> RepeatSubscriberId
 
  // Invalidates closure execution for the given subscriber(s)
  Repeat.invalidate(id: RepeatSubscriberId) -> Bool
  Repeat.invalidate(ids: [RepeatSubscriberId]) -> [Bool]
```  
  
1. 公共API使用单例来保持线程安全性，类型Objective-C的@synchronized。
2. 支持闭包函数在主线程中执行。  
  
  
完整代码下载： [GitHub](https://github.com/RobinChao/Repeat)