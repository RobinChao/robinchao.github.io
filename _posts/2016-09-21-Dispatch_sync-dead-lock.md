---
layout: post
author: Robin
title: Dispatch_sync(dispatch_get_main_queue,^{})死锁的问题
tags: 开发知识 iOS
categories:
  - 开发知识 
--- 

### Dispatch_sync(dispatch_get_main_queue,^{})死锁

`dispatch_sync`就是向特定的queue中插入一个`block`，等到这个block被执行了，再继续下面的任务，但是如果这个被安排任务的queue是`currentQueue`,那么将会成为死锁，因为sync会block currentqueue所以现在线程被block了，所以不会再继续执行下去。下面是搜到的英文解释。

> cdispatch_sync does two things:

> 1. queue a block
> 2. blocks the current thread until the block has finished running

> Given that the main thread is a serial queue (which means it uses only one thread), the following statement:
> dispatch_sync(dispatch_get_main_queue(), ^(){/*...*/});
> will cause the following events:

> 1. dispatch_sync queues the block in the main queue.
> 2. dispatch_sync blocks the thread of the main queue until the block finishes executing.
> 3. dispatch_sync waits forever because the thread where the block is supposed to run is blocked.

> The key to understanding this is that dispatch_sync does not execute blocks, it only queues them. Execution will happen on a  future iteration of the run loop.

还有，之所以要在mainqueue中去刷新界面，是因为有可能很多线程在后台运行，而UI的刷新永远是在主线程中去完成，所以UI的刷新如果被放到了其他的后台线程中，那么刷新就无法执行，所以必须要放在主线程中去做。