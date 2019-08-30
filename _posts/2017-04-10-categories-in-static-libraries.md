---
layout: post
title: 如何优雅的在静态库中使用的Categories
date: 2017-03-20 09:46:59 +0800
categories: 开发知识
tags: Categories iOS
---

在iOS开发的过程中，我们经常会将一些功能进行静态库封装，进而方便的在项目工程中使用，在静态库中会有各种的功能性代码，不可避免的会使用到*Categories*，而iOS静态库中对Categories的支持却并不简单，这里做一个总结。

### 链接到静态库

假设我们有如下的一个具有类别的静态库：

``` objc
 // MyStaticLib.h 
  @interface MyStaticLib：NSObject 
  +（void）doSomething; 
  @end 

  // MyStaticLib + Category.h 
  @interface MyStaticLib（Category） 
  +（void）sayHello; 
  @end
```

当Clang编译这两个文件时，它会生成两个不同的对象文件：MyStaticLib.o和MyStaticLib + Category.o。这些对象文件是实际包含我们的客户端应用程序将使用的可执行代码。

作为静态库编译的最后一步，所有生成的.o文件都打包在一个结果.a档案文件中。 此归档文件是静态库。

当我们在客户端应用程序中导入这些类的头文件并调用[MyStaticLib doSomething]时，在客户端应用程序中实际链接库中编译的代码的过程将留给链接器。

当链接器链接一个静态的lib时，它只将最小的所需对象从静态的libs导入最终的客户端应用程序。 任何未使用的对象文件都不会被包含。

链接类别问题发生在Objective C中，只有类名有符号。 方法由于Objc的活力，不出口符号。 可以使用nm来读取从生成的对象文件导出的符号。

``` sh
 > nm -gU MyStaticLib.o 
  00000000000007a8 S _OBJC_CLASS _ $ _ MyStaticLib 
  0000000000000780 S _OBJC_METACLASS _ $ _ MyStaticLib 

  > nm -gU MyStaticLib + Category.o 
```

从编译MyStaticLib.m生成的MyStaticLib.o正在导出一个objc类MyStaticLib及其元类。 另一方面，MyStaticLib + Category.o不会导出任何符号。


由于类别仅包含方法，它们不导出符号，这使得链接器无法决定何时将它们包含在库中，并且由于MyStaticLib + Category.o从未被包含在最终的应用程序产品中，所以调用[MyStaticLib sayHello]将导致无法识别的选择器运行时错误。


为了能够调用类别方法，我们需要告诉链接器来链接类别对象文件。 以下是在实践中得到的一些解决方案，可参考：

**解决方案1** 使用其他链接器标志（OTHER_LDFLAGS），添加**-Objc**，添加此标志将指示链接器链接所有链接的静态库中找到的所有目标c代码。

虽然这解决了这个问题，但它有两个主要的缺点： 首先，它增加最终二进制文件的大小，因为所有的类别，即使是我们实际上没有使用的类，也被添加到二进制文件中。 第二，它需要操纵客户端应用程序，这是图书馆用户必须记住的另一个步骤。

**解决方案2** 在库项目集中，将单个对象的前级链接**GENERATE_MASTER_OBJECT_FILE**设置为**YES**。 该标志将导致编译器将所有生成的对象文件加入到包含所有符号和代码的一个大对象文件中。 当客户端应用程序链接到静态库时，它将包含这个大对象文件作为一个单元。

您可以将此解决方案视为在库侧使用**-ObjC**。 虽然这可能会使您的客户端应用程序的大小更大，但不需要更改客户端应用程序上的构建设置。

**解决方案3** 确保类别代码和类代码最终在同一个目标文件中。将实际代码放在同一个.m文件中：

``` objc
  // MyStaticLib.m 
  @implementation MyStaticLib @end 
  @implementation MyStaticLib（Category）@end 
```

或者在类.m文件中导入类别.m:

``` objc
 // MyStaticLib.m 
  #import“MyStaticLib + Category.m” 
  @implementation MyStaticLib @end 
```

编译时，MyStaticLib.o生成的文件将同时包含类和类别的二进制代码。
当这个解决方案工作时，在.m文件中导入.m文件是可怕的。

**解决方案4** 在类别代码中添加假符号，并强制该符号加载到类文件中。

我们在MyStaticLib +类别中输入一个字符串MyStaticLibCategory:

``` objc
// MyStaticLib + Category.h 
  extern NSString * MyStaticLibCategory; 
  @interface MyStaticLib（Test） 

  // MyStaticLib + Category.m 
  NSString * MyStaticLibCategory; 
  @implementation MyStaticLib（Category） 
```

然后在MyStaticLib.m中使用该符号。

``` objc
 // anywhere in TestLib.m
  __attribute __（（used））static void importCategories（）{ 
  id x = MyStaticLibCategory; 
  } 
```

在上面的代码中，我们在类中添加了一个符号，然后在类实现中使用了这个符号。 这意味着，每当我们提取类的代码时，我们也会提取类别的代码。

## 个人建议

在实际使用的时候，个人建议使用**解决方案4**，这样做可以在内部或外部使用类别，而不是更改项目的构建设置，从而避免不同的设置导致某些类库无法使用的风险。
