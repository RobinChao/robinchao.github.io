---
layout: post
author: Robin
title: UICatalog Invalid Asset Error 如何解决
tags: 开发知识 iOS
categories:
  - 开发知识 
--- 

```
	CUICatalog: Invalid asset name supplied: (null), or invalid scale factor: 2.000000
```

在开发中，有时候会遇到这样的错误，但是并没有影响应用的运行。但是这样的错误会直接导致图片资源加载不上或者加载失效的问题。

其实这个错误的本质是：加载了一个空的UIImage。既然如此，那么可以直接使用Debug Command 来定位问题所在，然后重新部署图片资源了，具体如下：

iPhone Device condition:

`$r0 == nil`

Simulator condition:

`$arg3 == nil`

这样设置之后，启动应用后，如果再有如此的错误，Xcode就会停留在对应的代码行，就可以针对性的进行调整了。