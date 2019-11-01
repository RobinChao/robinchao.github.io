---
layout: post
author: Robin
title: 关于iOS App启动时间的那些事
tags: 开发知识 iOS
categories:
- 开发知识
cover: '/assets/start-time/cover.jpg'
---

在iOS应用程序的开发过程中，应用的启动时长可谓是影响应用程序用户体验的第一要素，过长的应用启动耗时，势必带来用户的长时间等待，直接让用户失去了对应用程序进一步体验的兴趣，影响应用程序在用户心中的印象。一般情况下，应用程序开发完成上线后，接下来就是针对架构、性能、业务进行进一步优化和调整的阶段，这个阶段也是检验一个iOS开发工程师内功的时候。

## iOS应用启动方式

iOS应用程序的启动整体分为**冷启动**和**热启动**两种方式，两种启动方式具有不同的启动触发条件，也是在不同的业务场景模式下，最终导致应用启动，进而延续业务的方式。

* **冷启动：**指的是当应用还没准备好运行时，必须从零开始加载和构建整个应用。包括设置屏幕底部的分栏菜单，确保用户是否被合适地登录，以及处理其他更多的事情。整个应用程序的入口是在*applicationDidFinishLaunching:withOptions:*方法中开始的。
* **热启动：**指的是应用已经运行但是在后台被挂起（比如用户点击了 `home` 健）。在这种情况下，应用通过 *applicationWillEnterForeground:* 接收到回到前台的事件，接着应用恢复。

> 另一种理解是，冷启动时App的进行不存在，系统需要为App分配进程等资源，以供App正确启动，而热启动时，App进程是存在的，只是App处于被挂起状态，热启动可以认为是App恢复形式的启动。

在应用启动时间的衡量和治理上，往往**冷启动**是重中之重，因为严格意义上，冷启动是包含热启动的（冷启动初始化应用程序并获取摘要，热启动仅获取摘要），另外，冷启动需要做的工作更多，其中包含了一些额外的初始化工作，也更加的耗时，因此，针对冷启动的治理更加有意义。

## 冷启动的定义

通常情况下，针对iOS的冷启动过程是从用户点击App图标开始到*applicationDidFinishLaunching:withOptions:*方法执行完毕为止，在这个过程中主要分为两个阶段：

* T1阶段：应用程序*main()*函数执行之前，即操作系统加载App可执行文件到内存，然后执行一系列的加载和链接等工作，最后执行至App的*main()*函数。
* T2阶段：*main()*函数执行之后，即从*main()*函数开始，到*applicationDidFinishLaunching:withOptions:*方法执行完毕。

![](/assets/start-time/code-start.jpg)

在T1阶段，通常也被称为**pre-main**阶段，在该阶段，主要的工作主角是操作系统，此时会执行如下几个工作：

![](/assets/start-time/pre-main)

在**pre-main**阶段做进行的各个任务，其主要工作也不尽相同，操作系统采用分而治之的策略，并顺序执行（可能会有并行的情况）。

|阶段|工作|
|-------|------|
|Load dylibs|Dyld从主执行文件的header中获取到需要加载的所依赖动态库列表，然后找到动态库所对应的每个dylib，而应用所依赖的dylib文件还可能依赖其他的dylib，所以所需要加载的动态列表是一个递归依赖的集合。|
|Rebase|Rebase是在Image内部调整指针的指向。在历史OS中，会把动态库加载到指定的地址，所有指针和数据对应的代码都是正确的，而在随着OS的演进，指针和数据所对应的地址空间演变成了随机化的方式，所以需要在原来地址的基础上根据随机的地址偏移量进行指向修正。|
|Bind|Bind是把指针正确地指向Image外部的内容，这些指向外部的指针被符号（symbol）名称绑定，dyld需要去符号表里进行查找，找到symbol对应的实现。|
|ObjC Setup|- 注册ObjC类（class registration）  - 把category的定义插入到方法列表（category registration） - 保证每个selector的唯一性（selector uniquing）|
|Initializers| - ObjC的+load()函数   - C++的构造函数属性函数等  - 非基本类型的C++静态全局变量的创建（通常是类或结构体）|

## pre-main阶段耗时统计

对于pre-main阶段，Xcode提供了针对上述各个阶段耗时统计的功能，只需要开发者为项目添加特殊的环境变量即可。针对pre-main耗时统计的环境变量有两个，分贝是**DYLD_PRINT_STATISTICS**和**DYLD_PRINT_STATISTICS_DETAILS**，前者是各个阶段的整体耗时统计，后者会输出一些更加详细的参数。

Xcode环境变量的添加位置在 *Product -> Scheme -> Edit Scheme -> Environment Variables*。

![](/assets/start-time/var-env.png)

> **DYLD_PRINT_STATISTICS**和**DYLD_PRINT_STATISTICS_DETAILS**的值设置为1表示开启，0表示关闭，默认为0.

设置之后，重启App，则会在Xcode的console中看到如下的统计输出：

```
Total pre-main time: 216.18 milliseconds (100.0%)
         dylib loading time:  61.15 milliseconds (28.2%)
        rebase/binding time: 126687488.9 seconds (372410141.8%)
            ObjC setup time:  25.85 milliseconds (11.9%)
           initializer time: 174.40 milliseconds (80.6%)
           slowest intializers :
             libSystem.B.dylib :  13.24 milliseconds (6.1%)
   libBacktraceRecording.dylib :   7.55 milliseconds (3.4%)
    libMainThreadChecker.dylib : 144.91 milliseconds (67.0%)
                              ...
```

当然我们也可以获取更详细的时间，只需将环境变量 **DYLD_PRINT_STATISTICS_DETAILS** 设为 1 就可以得到更加详细的信息：

```
total time: 966.57 milliseconds (100.0%)
  total images loaded:  334 (327 from dyld shared cache)
  total segments mapped: 21, into 370 pages
  total images loading time: 710.13 milliseconds (73.4%)
  total load time in ObjC:  20.68 milliseconds (2.1%)
  total debugger pause time: 472.96 milliseconds (48.9%)
  total dtrace DOF registration time:   0.15 milliseconds (0.0%)
  total rebase fixups:  17,943
  total rebase fixups time:   2.25 milliseconds (0.2%)
  total binding fixups: 457,972
  total binding fixups time: 188.15 milliseconds (19.4%)
  total weak binding fixups time:   0.01 milliseconds (0.0%)
  total redo shared cached bindings time: 201.78 milliseconds (20.8%)
  total bindings lazily fixed up: 0 of 0
  total time in initializers and ObjC +load:  45.17 milliseconds (4.6%)
                         libSystem.B.dylib :   5.18 milliseconds (0.5%)
               libBacktraceRecording.dylib :   5.59 milliseconds (0.5%)
                            CoreFoundation :   1.99 milliseconds (0.2%)
                libMainThreadChecker.dylib :  27.94 milliseconds (2.8%)
                    libLLVMContainer.dylib :   1.89 milliseconds (0.1%)
total symbol trie searches:    1109484
total symbol table binary searches:    0
total images defining weak symbols:  37
total images using weak symbols:  92
```

有了以上信息，就可以对pre-main阶段的时间消耗进行一个度量和优化了。那么除了上述两个环境变量外，Xcode还支持dyld的其他一些环境变量，如下：

|环境变量|描述说明|
|-------|-------|
|DYLD_PRINT_SEGMENTS|日志段映射|
|DYLD_PRINT_INITIALIZERS|日志图像初始化要求|
|DYLD_PRINT_BINDINGS|日志符号绑定|
|DYLD_PRINT_APIS|日志dyld API调用(例如，dlopen)|
|DYLD_PRINT_ENV|打印启动环境变量|
|DYLD_PRINT_OPTS|打印启动时命令行参数|
|DYLD_PRINT_LIBRARIES_POST_LAUNCH|日志库加载，但仅在main运行之后|
|DYLD_PRINT_LIBRARIES|日志库加载|
|DYLD_IMAGE_SUFFIX|首先搜索带有这个后缀的库|

## pre-main阶段的优化策略

从上可知，在pre-mian阶段，应用程序会执行dylib loading、rebase/binding、ObjC setup、initializers四个步骤，从每个阶段的主要工作分析得知：

|阶段|优化策略|
|-------|------|
|Load dylibs|1.尽量不使用内嵌（embedded）的dylib，加载内嵌dylib性能开销较大；2.合并已有的dylib和使用静态库（static archives），减少dylib的使用个数；3.懒加载dylib，但是要注意dlopen()可能造成一些问题，且实际上懒加载做的工作会更多|
|Rebase&Bind|1.减少ObjC类（class）、方法（selector）、分类（category）的数量；2.减少C++虚函数的的数量（创建虚函数表有开销）；3.使用Swift structs（内部做了优化，符号数量更少）|
|ObjC Setup|减少 Objective-C Class、Selector、Category 的数量，可以合并或者删减一些OC类|
|Initializers| 1.少在类的+load方法里做事情，尽量把这些事情推迟到+initiailize；2.减少构造器函数个数，在构造器函数里少做些事情；3.减少C++静态全局变量的个数）|

简单概括就是

1. 应用程序所依赖的动态库越多，启动越慢；
2. ObjC的类、方法越多，启动越慢；
3. Objc的+load()越多，或+load()中有过多的逻辑，启动越慢；
4. C的constructor函数越多，启动越慢；
5. C++静态对象越多，启动越慢。


以上是对iOS App启动（主要针对冷启动）耗时的一点总结性内容，在具体的项目开发过程中，开发人员应当追求更加简洁高效的代码实现，追求高内聚，低耦合的项目架构，并有意的进行代码优化，使得App的启动耗时控制在良好的范围内，只有高效的启动，才不会再App的第一关就被Pass掉，从而为后续的业务提供良好的开端。

## 工具集

在了解了如何进行pre-main阶段耗时治理的策略之后，你可以动手进行一系列的优化提升，这里不进行具体的代码展示，列出两个可使用的工具，有兴趣的小伙伴可以详细研读学习。

* [objc_cover](https://github.com/nst/objc_cover)：一款通过对Mach-O文件进行解读，从中找到方法列表后，根据是否有对方法进行引用判定方法是否有用，可以用来删除项目中无用的方法。
* [DynamicLoader](https://github.com/jinshilaoyao/DynamicLoader/tree/5ef77c9ce819304009cf6c610b526668962dfa17)：一款*项目自启动*技术，实现了可插拔的函数的注册和启动等。

## 参考资料

* [App Startup Time: Past, Present, and Future](https://developer.apple.com/videos/play/wwdc2017/413/)
* [手淘iOS性能优化探索](https://github.com/izhangxb/GMTC/blob/master/%E5%85%A8%E7%90%83%E7%A7%BB%E5%8A%A8%E6%8A%80%E6%9C%AF%E5%A4%A7%E4%BC%9AGMTC%202017%20PPT/%E6%89%8B%E6%B7%98iOS%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E6%8E%A2%E7%B4%A2%20.pdf)
* [探秘 Mach-O 文件](https://juejin.im/post/5ab47ca1518825611a406a39)