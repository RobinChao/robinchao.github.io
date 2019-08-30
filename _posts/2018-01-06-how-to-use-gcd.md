---
layout: post
author: Robin
title: 详谈 GCD（Grand Central Dispatch）的使用
tags: 开发知识 iOS
categories:
  - 开发知识
---

## GCD（Grand Central Dispatch）介绍

Grand Central Dispatch (GCD) 是 Apple 开发的一种多核编程的解决方法。该方法在 Mac OS X 10.6 Lion 中首次推出，并随后被引入到了 iOS4.0 中。GCD 是一个替代诸如 NSThread, NSOperationQueue, NSInvocationOperation 等技术的更高效和强大的技术。

GCD 属于系统级的线程管理技术，在 Dispatch queue 中执行相关任务的性能非常高。GCD 的源代码已经开源，感兴趣的可以参考[Grand Central Dispatch](https://apple.github.io/swift-corelibs-libdispatch/)。 GCD 中的FIFO队列称为 dispatch queue，以用来保证先进入队列的任务先得到执行。

### GCD 简述

* 和Operation queue相同，都是基于队列的并发编程API，均是通过集中管理、协同使用的线程池。
* GCD具有5个不同的队列：
    1. 运行在主线程中的 Main queue
    2. 三个不同优先级的队列（High Priority Queue，Default Priority Queue，Low Priority Queue）。
    3. 更低优先级的后台队列 Background Priority Queue，主要用于I/O。
* 用户可自定义创建队列：串行或并行队列。
* 具体的操作时在多线程上还是单线程上，主要依据队列的类型和执行方法，并行队列异步执行在多线程，并行队列同步执行只会在单线程（主线程）执行。

## GCD 基本的概念

* 标准队列

标准队列是指GCD预定义的队列，在iOS系统中主要有两个：

```swift
//全局队列，一个并行的队列
dispatch_get_global_queue
//主队列，主线程中的唯一的队列，一个串行队列
dispatch_get_main_queue
```

* 自定义队列

用户可以自定构建队列，并设置队列是并行还是串行：

```swift
//串行队列
dispatch_queue_create("com.robin.serialqueue", DISPATCH_QUEUE_SERIAL)
//并行队列
dispatch_queue_create("com.robin.concurrentqueue", DISPATCH_QUEUE_CONCURRENT)
```

* 同步\异步线程创建

用户也可以根据需要自行构建同步\异步线程：

```swift
//同步线程
dispatch_sync(..., ^(block))
//异步线程
dispatch_async(..., ^(block))
```

## 队列（dispatch queue）

* Serial（串行队列）：又叫做 private dispatch queues，同时只执行一个任务（Task）。Serial queue 常用语同步访问特定的资源或数据，当创建了多个 Serial queue 时，虽然各自是同步的，但是 Serial queue 之间是并发执行的。

* Main dispatch queue: 全局可用的 Serial queue，在应用程序的主线程上执行。

* Concurrent（并行队列）：又叫做 global dispatch queues，可以并发的执行多个任务，但是执行的顺序是随机的。iOS 系统提供了四个全局并发队列，这四个队列有着对应的优先级，用户是不能创建全局队列的，只能获取，如下：

```swift 
dipatch_queue_t queue;
queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH,0);
```

* Custom（自定义队列）：用户可以自定义队列，使用 *dispatch_queue_create* 函数，并附带队列名和队列类型参数，其中队列类型默认是NULL，代表DISPATCH_QUEUE_SERIAL串行队列，可以使用参数DISPATCH_QUEUE_CONCURRENT设置并行队列。

```swift
dispatch_queue_t queue
queue = dispatch_queue_create("com.robin.learning.concurrentqueue", DISPATCH_QUEUE_CONCURRENT);
```

* 自定义队列的优先级：在自定义队列的时候，可以设置队列的优先级，使用*dipatch_queue_attr_make_with_qos_class*或者*dispatch_set_target_queue*方法来设置，如下：

```swift
//dipatch_queue_attr_make_with_qos_class
dispatch_queue_attr_t attr = dispatch_queue_attr_make_with_qos_class(DISPATCH_QUEUE_SERIAL, QOS_CLASS_UTILITY, -1);
dispatch_queue_t queue = dispatch_queue_create("com.robin.learning.qosqueue", attr);
//dispatch_set_target_queue
dispatch_queue_t queue = dispatch_queue_create("com.robin.learning.settargetqueue",NULL); //需要设置优先级的queue
dispatch_queue_t referQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0); //参考优先级
dispatch_set_target_queue(queue, referQueue); //设置queue和referQueue的优先级一样
```

* dispatch_set_target_queue：此方法不仅能够设置队列优先级，还可以设置队列的层级体系，比如让多个串行队列和并行队列在统一的一个串行队列里执行，如下：

```swift
dispatch_queue_t serialQueue = dispatch_queue_create("com.robin.learning.serialqueue", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t firstQueue = dispatch_queue_create("com.robin.learning.firstqueue", DISPATCH_QUEUE_SERIAL);
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.robin.learning.secondqueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_set_target_queue(firstQueue, serialQueue);
dispatch_set_target_queue(concurrentQueue, serialQueue);
dispatch_async(firstQueue, ^{
    NSLog(@"1");
    [NSThread sleepForTimeInterval:3.f];
});
dispatch_async(concurrentQueue, ^{
    NSLog(@"2");
    [NSThread sleepForTimeInterval:2.f];
});
dispatch_async(concurrentQueue, ^{
    NSLog(@"3");
    [NSThread sleepForTimeInterval:1.f];
});
```

## 队列的类型

在iOS中，队列本身默认是串行的，只能执行一个单独的block，但是队列也可以是并行的，同一时间执行多个block。

在创建队列时，我们通常使用**dispatch_queue_create**函数：

```objc
- (id)init;
{
     self = [super init];
     if (self != nil) {
          NSString *label = [NSString stringWithFormat:@"%@.isolation.%p", [self class], self];
          self.isolationQueue = dispatch_queue_create([label UTF8String], 0);
          label = [NSString stringWithFormat:@"%@.work.%p", [self class], self];
          self.workQueue = dispatch_queue_create([label UTF8String], 0);
     }
     return self;
}
```

iOS中的公开5个队列：主队列（main queue）、四个通用调度队列以及用户定制的队列。对于四个通用调度队列，分别为：

* QOS_CLASS_USER_INTERACTIVE：user interactive 等级表示任务需要被立即执行已提供最好的用户体验，更新UI或者相应事件等，这个等级最好越小规模越好。

* QOS_CLASS_USER_INITIATED：user initiated等级表示任务由UI发起异步执行。适用场景是需要及时结果同时又可以继续交互的时候。

* QOS_CLASS_UTILITY：utility等级表示需要长时间运行的任务，伴有用户可见进度指示器。经常会用来做计算，I/O，网络，持续的数据填充等任务。

* QOS_CLASS_BACKGROUND：background等级表示用户不会察觉的任务，使用它来处理预加载，或者不需要用户交互和对时间不敏感的任务。

一个典型的实例就是在后台加载图片：

```swift
override func viewDidLoad() {
     super.viewDidLoad()
     dispatch_async(dispatch_get_global_queue(Int(QOS_CLASS_USER_INITIATED.value), 0)) { // 将工作从主线程转移到全局队列中，这是dispatch_async调用，异步提交保证调用线程会继续执行下去，这样viewDidLoad在主线程上能够更早完成，
          let overlayImage = self.faceOverlayImageFromImage(self.image)
          dispatch_async(dispatch_get_main_queue()) { // 新图完成，把一个闭包加入主线程用来更新UIImageView，只有在主线程能操作UIKit。
               self.fadeInNewImage(overlayImage) // 更新UI
          }
     }
}
```

### 队列类型的使用

那么具体在操作中，什么时候使用什么类型的队列呢？通常有如下的规则：

* 主队列：主队列通常是其他队列中有任务完成，需要更新UI时，例如使用延后执行*dispatch_after*的场景。

* 并发队列：并发队列通常用来执行和UI无关的后台任务，但是有时还需要保持数据或者读写的同步，会使用dispatch_sync或者dispatch_barrier_sync 同步。

* 自定义顺序队列：顺序执行后台任务并追踪它时。这样做同时只有一个任务在执行可以防止资源竞争。通常会使用dipatch barriers解决读写锁问题，或者使用dispatch groups解决锁问题。


可以使用下面的方法简化QoS等级参数的写法：

```swift
var GlobalMainQueue: dispatch_queue_t {
     return dispatch_get_main_queue()
}
var GlobalUserInteractiveQueue: dispatch_queue_t {
     return dispatch_get_global_queue(Int(QOS_CLASS_USER_INTERACTIVE.value), 0)
}
var GlobalUserInitiatedQueue: dispatch_queue_t {
     return dispatch_get_global_queue(Int(QOS_CLASS_USER_INITIATED.value), 0)
}
var GlobalUtilityQueue: dispatch_queue_t {
     return dispatch_get_global_queue(Int(QOS_CLASS_UTILITY.value), 0)
}
var GlobalBackgroundQueue: dispatch_queue_t {
     return dispatch_get_global_queue(Int(QOS_CLASS_BACKGROUND.value), 0)
}
//使用起来就是这样，易读而且容易看出在使用哪个队列
dispatch_async(GlobalUserInitiatedQueue) {
     let overlayImage = self.faceOverlayImageFromImage(self.image)
     dispatch_async(GlobalMainQueue) {
          self.fadeInNewImage(overlayImage)
     }
}
```

## dispatch_once用法

dispatch_once_t要是全局或static变量，保证dispatch_once_t只有一份实例。

```objc
+ (UIColor *)boringColor;
{
     static UIColor *color;
     //只运行一次
     static dispatch_once_t onceToken;
     dispatch_once(&onceToken, ^{
          color = [UIColor colorWithRed:0.380f green:0.376f blue:0.376f alpha:1.000f];
     });
     return color;
}
```

## dispatch_async

设计一个异步的API时调用dispatch_async()，这个调用放在API的方法或函数中做。让API的使用者有一个回调处理队列。

```objc
- (void)processImage:(UIImage *)image completionHandler:(void(^)(BOOL success))handler;
{
     dispatch_async(self.isolationQueue, ^(void){
          // do actual processing here
          dispatch_async(self.resultQueue, ^(void){
               handler(YES);
          });
     });
}
```

可以避免界面会被一些耗时的操作卡死，比如读取网络数据，大数据IO，还有大量数据的数据库读写，这时需要在另一个线程中处理，然后通知主线程更新界面，GCD使用起来比NSThread和NSOperation方法要简单方便。

```objc
//代码框架
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
     // 耗时的操作
     dispatch_async(dispatch_get_main_queue(), ^{
          // 更新界面
     });
});
//下载图片的示例
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
     NSURL * url = [NSURL URLWithString:@"http://avatar.csdn.net/2/C/D/1_totogo2010.jpg"];
     NSData * data = [[NSData alloc]initWithContentsOfURL:url];
     UIImage *image = [[UIImage alloc]initWithData:data];
     if (data != nil) {
          dispatch_async(dispatch_get_main_queue(), ^{
               self.imageView.image = image;
          });
     }
});
```

## dispatch_after延后执行

dispatch_after只是延时提交block，不是延时立刻执行。

```objc
- (void)foo
{
     double delayInSeconds = 2.0;
     dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t) (delayInSeconds * NSEC_PER_SEC));
     dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
          [self bar];
     });
}
```

范例，实现一个推迟出现弹出框提示，比如说提示用户评价等功能。

```swift
func showOrHideNavPrompt() {
     let delayInSeconds = 1.0
     let popTime = dispatch_time(DISPATCH_TIME_NOW,
          Int64(delayInSeconds * Double(NSEC_PER_SEC))) // 在这里声明推迟的时间
     dispatch_after(popTime, GlobalMainQueue) { // 等待delayInSeconds将闭包异步到主队列
          let count = PhotoManager.sharedManager.photos.count
          if count > 0 {
               self.navigationItem.prompt = nil
          } else {
               self.navigationItem.prompt = "Add photos with faces to Googlyify them!"
          }
     }
}
```

例子中的dispatch time的参数，可以先看看函数原型

```objc
dispatch_time_t dispatch_time ( dispatch_time_t when, int64_t delta );
```

第一个参数为DISPATCH_TIME_NOW表示当前。第二个参数的delta表示纳秒，一秒对应的纳秒为1000000000，系统提供了一些宏来简化：

```objc
#define NSEC_PER_SEC 1000000000ull //每秒有多少纳秒
#define USEC_PER_SEC 1000000ull    //每秒有多少毫秒
#define NSEC_PER_USEC 1000ull      //每毫秒有多少纳秒
```

这样如果要表示一秒就可以这样写

```objc
dispatch_time(DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC);
dispatch_time(DISPATCH_TIME_NOW, 1000 * USEC_PER_SEC);
dispatch_time(DISPATCH_TIME_NOW, USEC_PER_SEC * NSEC_PER_USEC);
```

**dispatch_barrier_async使用Barrier Task方法Dispatch Barrier解决多线程并发读写同一个资源发生死锁**

Dispatch Barrier确保提交的闭包是指定队列中在特定时段唯一在执行的一个。在所有先于Dispatch Barrier的任务都完成的情况下这个闭包才开始执行。轮到这个闭包时barrier会执行这个闭包并且确保队列在此过程不会执行其它任务。闭包完成后队列恢复。需要注意dispatch_barrier_async只在自己创建的队列上有这种作用，在全局并发队列和串行队列上，效果和dispatch_sync一样。

```objc
//创建队列
self.isolationQueue = dispatch_queue_create([label UTF8String], DISPATCH_QUEUE_CONCURRENT);
//改变setter
- (void)setCount:(NSUInteger)count forKey:(NSString *)key
{
     key = [key copy];
     //确保所有barrier都是async异步的
     dispatch_barrier_async(self.isolationQueue, ^(){
          if (count == 0) {
               [self.counts removeObjectForKey:key];
          } else {
               self.counts[key] = @(count);
          }
     });
}
- (void)dispatchBarrierAsyncDemo {
    //防止文件读写冲突，可以创建一个串行队列，操作都在这个队列中进行，没有更新数据读用并行，写用串行。
    dispatch_queue_t dataQueue = dispatch_queue_create("com.starming.gcddemo.dataqueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(dataQueue, ^{
        [NSThread sleepForTimeInterval:2.f];
        NSLog(@"read data 1");
    });
    dispatch_async(dataQueue, ^{
        NSLog(@"read data 2");
    });
    //等待前面的都完成，在执行barrier后面的
    dispatch_barrier_async(dataQueue, ^{
        NSLog(@"write data 1");
        [NSThread sleepForTimeInterval:1];
    });
    dispatch_async(dataQueue, ^{
        [NSThread sleepForTimeInterval:1.f];
        NSLog(@"read data 3");
    });
    dispatch_async(dataQueue, ^{
        NSLog(@"read data 4");
    });
}
```

Swift示例：

```swift
//使用dispatch_queue_create初始化一个并发队列。第一个参数遵循反向DNS命名习惯，方便描述，第二个参数是指出是并发还是顺序。
private let concurrentPhotoQueue = dispatch_queue_create(
"com.raywenderlich.GooglyPuff.photoQueue", DISPATCH_QUEUE_CONCURRENT)
func addPhoto(photo: Photo) {
     dispatch_barrier_async(concurrentPhotoQueue) { // 将写操作加入到自定义的队列。开始执行时这个就是队列中唯一的一个在执行的任务。
          self._photos.append(photo) // barrier能够保障不会和其他任务同时进行。
          dispatch_async(GlobalMainQueue) { // 涉及到UI所以这个通知应该在主线程中，所以分派另一个异步任务到主队列中。
               self.postContentAddedNotification()
          }
     }
}
//上面是解决了写可能发生死锁，下面是使用dispatch_sync解决读时可能会发生的死锁。
var photos: [Photo] {
     var photosCopy: [Photo]!
     dispatch_sync(concurrentPhotoQueue) { // 同步调度到concurrentPhotoQueue队列执行读操作
          photosCopy = self._photos // 保存
     }
     return photosCopy
}
//这样读写问题都解决了。
```

都用异步处理避免死锁，异步的缺点在于调试不方便，但是比起同步容易产生死锁这个副作用还算小的。

## dispatch_apply进行快速迭代

类似for循环，但是在并发队列的情况下dispatch_apply会并发执行block任务。

```objc
for (size_t y = 0; y < height; ++y) {
     for (size_t x = 0; x < width; ++x) {
          // Do something with x and y here
     }
}
//因为可以并行执行，所以使用dispatch_apply可以运行的更快
- (void)dispatchApplyDemo {
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_apply(10, concurrentQueue, ^(size_t i) {
        NSLog(@"%zu",i);
    });
    NSLog(@"The end"); //这里有个需要注意的是，dispatch_apply这个是会阻塞主线程的。这个log打印会在dispatch_apply都结束后才开始执行
}
```

dispatch_apply能避免线程爆炸，因为GCD会管理并发。

```objc
- (void)dealWiththreadWithMaybeExplode:(BOOL)explode {
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
    if (explode) {
        //有问题的情况，可能会死锁
        for (int i = 0; i < 999 ; i++) {
            dispatch_async(concurrentQueue, ^{
                NSLog(@"wrong %d",i);
                //do something hard
            });
        }
    } else {
        //会优化很多，能够利用GCD管理
        dispatch_apply(999, concurrentQueue, ^(size_t i){
            NSLog(@"correct %zu",i);
            //do something hard
        });
    }
}
```
示例：

```swift
func downloadPhotosWithCompletion(completion: BatchPhotoDownloadingCompletionClosure?) {
     var storedError: NSError!
     var downloadGroup = dispatch_group_create()
     let addresses = [OverlyAttachedGirlfriendURLString,
          SuccessKidURLString,
          LotsOfFacesURLString]
     dispatch_apply(UInt(addresses.count), GlobalUserInitiatedQueue) {
          i in
          let index = Int(i)
          let address = addresses[index]
          let url = NSURL(string: address)
          dispatch_group_enter(downloadGroup)
          let photo = DownloadPhoto(url: url!) {
               image, error in
               if let error = error {
                    storedError = error
               }
               dispatch_group_leave(downloadGroup)
          }
          PhotoManager.sharedManager.addPhoto(photo)
     }
     dispatch_group_notify(downloadGroup, GlobalMainQueue) {
          if let completion = completion {
               completion(error: storedError)
          }
     }
}
```

## Block组合Dispatch_groups

dispatch groups是专门用来监视多个异步任务。dispatch_group_t实例用来追踪不同队列中的不同任务。

当group里所有事件都完成GCD API有两种方式发送通知，第一种是dispatch_group_wait，会阻塞当前进程，等所有任务都完成或等待超时。第二种方法是使用dispatch_group_notify，异步执行闭包，不会阻塞。

第一种使用dispatch_group_wait的swift的例子：

```swift
func downloadPhotosWithCompletion(completion: BatchPhotoDownloadingCompletionClosure?) {
     dispatch_async(GlobalUserInitiatedQueue) { // 因为dispatch_group_wait会租塞当前进程，所以要使用dispatch_async将整个方法要放到后台队列才能够保证主线程不被阻塞
          var storedError: NSError!
          var downloadGroup = dispatch_group_create() // 创建一个dispatch group
          for address in [OverlyAttachedGirlfriendURLString,
               SuccessKidURLString,
               LotsOfFacesURLString]
          {
               let url = NSURL(string: address)
               dispatch_group_enter(downloadGroup) // dispatch_group_enter是通知dispatch group任务开始了，dispatch_group_enter和dispatch_group_leave是成对调用，不然程序就崩溃了。
               let photo = DownloadPhoto(url: url!) {
                    image, error in
                    if let error = error {
                         storedError = error
                    }
                    dispatch_group_leave(downloadGroup) // 保持和dispatch_group_enter配对。通知任务已经完成
               }
               PhotoManager.sharedManager.addPhoto(photo)
          }
          dispatch_group_wait(downloadGroup, DISPATCH_TIME_FOREVER) // dispatch_group_wait等待所有任务都完成直到超时。如果任务完成前就超时了，函数会返回一个非零值，可以通过返回值判断是否超时。也可以用DISPATCH_TIME_FOREVER表示一直等。
          dispatch_async(GlobalMainQueue) { // 这里可以保证所有图片任务都完成，然后在main queue里加入完成后要处理的闭包，会在main queue里执行。
               if let completion = completion { // 执行闭包内容
                    completion(error: storedError)
               }
          }
     }
}
```

OC例子:

```objc
- (void)dispatchGroupWaitDemo {
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
    dispatch_group_t group = dispatch_group_create();
    //在group中添加队列的block
    dispatch_group_async(group, concurrentQueue, ^{
        [NSThread sleepForTimeInterval:2.f];
        NSLog(@"1");
    });
    dispatch_group_async(group, concurrentQueue, ^{
        NSLog(@"2");
    });
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    NSLog(@"go on");
}
```

第二种使用dispatch_group_notify的swift的例子：

```swift
func downloadPhotosWithCompletion(completion: BatchPhotoDownloadingCompletionClosure?) {
     // 不用加dispatch_async，因为没有阻塞主进程
     var storedError: NSError!
     var downloadGroup = dispatch_group_create()
     for address in [OverlyAttachedGirlfriendURLString,
          SuccessKidURLString,
          LotsOfFacesURLString]
     {
          let url = NSURL(string: address)
          dispatch_group_enter(downloadGroup)
          let photo = DownloadPhoto(url: url!) {
               image, error in
               if let error = error {
                    storedError = error
               }
               dispatch_group_leave(downloadGroup)
          }
          PhotoManager.sharedManager.addPhoto(photo)
     }
     dispatch_group_notify(downloadGroup, GlobalMainQueue) { // dispatch_group_notify和dispatch_group_wait的区别就是是异步执行闭包的，当dispatch groups中没有剩余的任务时闭包才执行。这里是指明在主队列中执行。
          if let completion = completion {
               completion(error: storedError)
          }
     }
}
```

OC例子：

```objc
//dispatch_group_notify
- (void)dispatchGroupNotifyDemo {
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, concurrentQueue, ^{
        NSLog(@"1");
    });
    dispatch_group_async(group, concurrentQueue, ^{
        NSLog(@"2");
    });
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"end");
    });
    NSLog(@"can continue");
}
//dispatch_group_wait
- (void)dispatchGroupWaitDemo {
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
    dispatch_group_t group = dispatch_group_create();
    //在group中添加队列的block
    dispatch_group_async(group, concurrentQueue, ^{
        [NSThread sleepForTimeInterval:2.f];
        NSLog(@"1");
    });
    dispatch_group_async(group, concurrentQueue, ^{
        NSLog(@"2");
    });
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    NSLog(@"can continue");
}
```

如何对现有API使用dispatch_group_t:

```objc
//给Core Data的-performBlock:添加groups。组合完成任务后使用dispatch_group_notify来运行一个block即可。
- (void)withGroup:(dispatch_group_t)group performBlock:(dispatch_block_t)block
{
     if (group == NULL) {
          [self performBlock:block];
     } else {
          dispatch_group_enter(group);
          [self performBlock:^(){
               block();
               dispatch_group_leave(group);
          }];
     }
}
//NSURLConnection也可以这样做
+ (void)withGroup:(dispatch_group_t)group
     sendAsynchronousRequest:(NSURLRequest *)request
     queue:(NSOperationQueue *)queue
     completionHandler:(void (^)(NSURLResponse*, NSData*, NSError*))handler
{
     if (group == NULL) {
          [self sendAsynchronousRequest:request
               queue:queue
               completionHandler:handler];
     } else {
          dispatch_group_enter(group);
          [self sendAsynchronousRequest:request
                    queue:queue
                    completionHandler:^(NSURLResponse *response, NSData *data, NSError *error){
               handler(response, data, error);
               dispatch_group_leave(group);
          }];
     }
}
```
注意事项

* dispatch_group_async等价于dispatch_group_enter() 和 dispatch_group_leave()的组合。
* dispatch_group_enter() 必须运行在 dispatch_group_leave() 之前。
* dispatch_group_enter() 和 dispatch_group_leave() 需要成对出现的


## Dispatch Block

#### 队列执行任务都是block的方式

* 创建block

```objc
- (void)createDispatchBlock {
    //normal way
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue",DISPATCH_QUEUE_CONCURRENT);
    dispatch_block_t block = dispatch_block_create(0, ^{
        NSLog(@"run block");
    });
    dispatch_async(concurrentQueue, block);
    //QOS way
    dispatch_block_t qosBlock = dispatch_block_create_with_qos_class(0, QOS_CLASS_USER_INITIATED, -1, ^{
        NSLog(@"run qos block");
    });
    dispatch_async(concurrentQueue, qosBlock);
}
```

* dispatch_block_wait：可以根据dispatch block来设置等待时间，参数DISPATCH_TIME_FOREVER会一直等待block结束

```objc
- (void)dispatchBlockWaitDemo {
    dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
    dispatch_block_t block = dispatch_block_create(0, ^{
        NSLog(@"star");
        [NSThread sleepForTimeInterval:5.f];
        NSLog(@"end");
    });
    dispatch_async(serialQueue, block);
    //设置DISPATCH_TIME_FOREVER会一直等到前面任务都完成
    dispatch_block_wait(block, DISPATCH_TIME_FOREVER);
    NSLog(@"ok, now can go on");
}
```

* dispatch_block_notify：可以监视指定dispatch block结束，然后再加入一个block到队列中。三个参数分别为，第一个是需要监视的block，第二个参数是需要提交执行的队列，第三个是待加入到队列中的block

``` objc
- (void)dispatchBlockNotifyDemo {
    dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
    dispatch_block_t firstBlock = dispatch_block_create(0, ^{
        NSLog(@"first block start");
        [NSThread sleepForTimeInterval:2.f];
        NSLog(@"first block end");
    });
    dispatch_async(serialQueue, firstBlock);
    dispatch_block_t secondBlock = dispatch_block_create(0, ^{
        NSLog(@"second block run");
    });
    //first block执行完才在serial queue中执行second block
    dispatch_block_notify(firstBlock, serialQueue, secondBlock);
}
```

* dispatch_block_cancel：iOS8后GCD支持对dispatch block的取消

```objc
- (void)dispatchBlockCancelDemo {
    dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
    dispatch_block_t firstBlock = dispatch_block_create(0, ^{
        NSLog(@"first block start");
        [NSThread sleepForTimeInterval:2.f];
        NSLog(@"first block end");
    });
    dispatch_block_t secondBlock = dispatch_block_create(0, ^{
        NSLog(@"second block run");
    });
    dispatch_async(serialQueue, firstBlock);
    dispatch_async(serialQueue, secondBlock);
    //取消secondBlock
    dispatch_block_cancel(secondBlock);
}
```

### 使用dispatch block object（调度块）在任务执行前进行取消


dispatch block object可以为队列中的对象设置。

示例，下载图片中途进行取消：

```swift
func downloadPhotosWithCompletion(completion: BatchPhotoDownloadingCompletionClosure?) {
     var storedError: NSError!
     let downloadGroup = dispatch_group_create()
     var addresses = [OverlyAttachedGirlfriendURLString,
          SuccessKidURLString,
          LotsOfFacesURLString]
     addresses += addresses + addresses // 扩展address数组，复制3份
     var blocks: [dispatch_block_t] = [] // 一个保存block的数组
     for i in 0 ..< addresses.count {
          dispatch_group_enter(downloadGroup)
          let block = dispatch_block_create(DISPATCH_BLOCK_INHERIT_QOS_CLASS) { // 创建一个block，block的标志是DISPATCH_BLOCK_INHERIT_QOS_CLASS
               let index = Int(i)
               let address = addresses[index]
               let url = NSURL(string: address)
               let photo = DownloadPhoto(url: url!) {
                    image, error in
                    if let error = error {
                         storedError = error
                    }
                    dispatch_group_leave(downloadGroup)
               }
               PhotoManager.sharedManager.addPhoto(photo)
          }
          blocks.append(block)
          dispatch_async(GlobalMainQueue, block) // 把这个block放到GlobalMainQueue上异步调用。因为全局队列是一个顺序队列所以方便取消对象block，同时可以保证下载任务在downloadPhotosWithCompletion返回后才开始执行。
     }
     for block in blocks[3 ..< blocks.count] {
          let cancel = arc4random_uniform(2) // 随机返回一个整数，会返回0或1
          if cancel == 1 {
               dispatch_block_cancel(block) // 如果是1就取消block，这个只能发生在block还在队列中并没有开始的情况下。因为把block已经放到了GlobalMainQueue中，所以这个地方会先执行，执行完了才会执行block。
               dispatch_group_leave(downloadGroup) // 因为已经dispatch_group_enter了，所以取消时也要将其都leave掉。
          }
     }
     dispatch_group_notify(downloadGroup, GlobalMainQueue) {
          if let completion = completion {
               completion(error: storedError)
          }
     }
}
```

### Dispatch IO 文件操作

dispatch io读取文件的方式类似于下面的方式，多个线程去读取文件的切片数据，对于大的数据文件这样会比单线程要快很多。

```swift
dispatch_async(queue,^{/*read 0-99 bytes*/});
dispatch_async(queue,^{/*read 100-199 bytes*/});
dispatch_async(queue,^{/*read 200-299 bytes*/});
```
* dispatch_io_create：创建dispatch io
* dispatch_io_set_low_water：指定切割文件大小
* dispatch_io_read：读取切割的文件然后合并。


苹果系统日志API里用到了这个技术，可以在[这里](https://github.com/Apple-FOSS-Mirror/Libc/blob/2ca2ae74647714acfc18674c3114b1a5d3325d7d/gen/asl.c)查看。


```swift
pipe_q = dispatch_queue_create("PipeQ", NULL);
//创建
pipe_channel = dispatch_io_create(DISPATCH_IO_STREAM, fd, pipe_q, ^(int err){
    close(fd);
});
*out_fd = fdpair[1];
//设置切割大小
dispatch_io_set_low_water(pipe_channel, SIZE_MAX);
dispatch_io_read(pipe_channel, 0, SIZE_MAX, pipe_q, ^(bool done, dispatch_data_t pipedata, int err){
    if (err == 0)
    {
        size_t len = dispatch_data_get_size(pipedata);
        if (len > 0)
        {
            //对每次切块数据的处理
            const char *bytes = NULL;
            char *encoded;
            uint32_t eval;
            dispatch_data_t md = dispatch_data_create_map(pipedata, (const void **)&bytes, &len);
            encoded = asl_core_encode_buffer(bytes, len);
            asl_msg_set_key_val(aux, ASL_KEY_AUX_DATA, encoded);
            free(encoded);
            eval = _asl_evaluate_send(NULL, (aslmsg)aux, -1);
            _asl_send_message(NULL, eval, aux, NULL);
            asl_msg_release(aux);
            dispatch_release(md);
        }
    }
    if (done)
    {
        //semaphore +1使得不需要再等待继续执行下去。
        dispatch_semaphore_signal(sem);
        dispatch_release(pipe_channel);
        dispatch_release(pipe_q);
    }
});
```


此内容就到这里了。其他的技术点将会陆续放出，GCD是个好东西，但是在使用的时候一定要理清楚其含义，否则很容易出现不可调试的问题等。