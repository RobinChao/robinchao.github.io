---
layout: post
author: Robin
title: Runtime剖析01 --- 基本数据结构：objc_object & objc_class
tags: Runtime
---

众所周知，Objective-C语言是一门动态性很强的语言，与C、C++等语言有着很大的不同。Objective-C语言的动态性基本上都是由Runtime机制进行支撑和实现的，Runtime的实现，融合了C、C++，以及汇编语言。

## 什么是Runtime？

C、C++等静态语言中的各种数据结构都是在编译期已经决定了，不能被修改，而Objective-C作为动态性语言，在程序的运行期，可以动态修改一个类的结构，例如修改方法的实现，变量的绑定等等。

Objective-C语言作为动态语言，将原本编译期决定的事情推迟到运行期，仅仅采用编译器是无法完成的，因此就需要运行期有一套自己的运行时系统，而这个系统就是Runtime，也是Runtime存在的意义，以及Objective-C语言运行框架的基石。

在实际的开发中，与Runtime交互的情况大致有三种：

1. **Objective-C源码：**大多数情况下，开发者采用的都是直接使用Objective-C语言进行编码，而在Objective-C语言源码的背后，都是由Runtime进行底层支撑，Objective-C语言中所使用的数据类型，在Runtime中都有对应的C语言结构体，甚至汇编语言的实现等。
2. **通过NSObject：**在Cocoa中，大部分的类都继承自NSObject，而NSObject的定义是Runtime决定的。以及NSObject中的大多数方法，都是运行时动态决定的，背后其实是Runtime对应数据结构的支持。例如常用的`isKindOfClass`和`isMemberOfClass`检查类是否属于指定的Class的继承体系中；`responderToSelector` 检查对象是否能响应指定的消息；`conformsToProtocol` 检查对象是否遵循某个协议；`methodForSelector`返回指定方法实现的地址等。
3. **直接调用Runtime API：**Runtime是一个由一系列函数和数据机构组成，具有公共接口的动态共享库。很多函数的功能和Objective-C语言中的含有具有同等的功能。一般情况下不会直接访问Runtime的API，但是当有一些底层的需求需要实现时，例如为现有类动态添加属性等。[Objective-C Runtime Reference](https://developer.apple.com/documentation/objectivec/objective_c_runtime?language=objc)

**Objective-C语言中的各种黑魔法，其实都是在Runtime的基础上，对底层数据机构的各类应用。**

## NSObject解析

在Objective-C中，几乎所有的类的基类，都是**NSObject**。因此要深入了解Objective-C类的相关结构，先从**NSObject**类开始。

在iOS SDK中，对于NSObject类的定义如下：

```objectivec
@interface NSObject <NSObject> {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wobjc-interface-ivars"
    Class isa  OBJC_ISA_AVAILABILITY;
#pragma clang diagnostic pop
}
```

**NSObject**类仅有一个实例变量**isa**，并且遵循**NSObject**协议。先说说**NSObject**协议，在该协议中，定义了NSObject类的一些通用方法，例如`performSelector：`、`isMemberOfClass`、`superclass`等等，使用协议去定义通用的一些方法，也让类的扩展更加的容易。

**NSObject**类的变量仅有**Class isa**，变量的类型**Class**定义如下：

```c
typedef struct objc_class *Class;
```

**Class**本质是指向**objc_class结构体**的指针，而**objc_class结构体**的定义如下：

```c
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

    // 省略其他方法
    ...
}
```

## objc_class

可以看到**objc_class**继承自**objc_object**，即在Runtime中，class本质也是一个对象。在**objc_class**结构体的定义中，有三个成员变量：

* **Class superclass：** 表示当前类的父类对象，类型同样是**Class**；
* **cache_t cache：** Objective-C方法调用的优化结构体。对应的数据结构如下：

```c
struct cache_t {
	struct bucket_t *buckets();
    mask_t mask();
    mask_t occupied();
    void incrementOccupied();
    void setBucketsAndMask(struct bucket_t *newBuckets, mask_t newMask);
    void initializeToEmpty();

    unsigned capacity();
    bool isConstantEmptyCache();
    bool canBeFreed();

    // 省略其他方法
    ...
}

struct bucket_t {
	explicit_atomic<uintptr_t> _imp;
    explicit_atomic<SEL> _sel;
public:
	inline SEL sel() const { return _sel.load(memory_order::memory_order_relaxed); }

    inline IMP imp(Class cls) const {
        uintptr_t imp = _imp.load(memory_order::memory_order_relaxed);
        if (!imp) return nil;
#if CACHE_IMP_ENCODING == CACHE_IMP_ENCODING_PTRAUTH
        SEL sel = _sel.load(memory_order::memory_order_relaxed);
        return (IMP)
            ptrauth_auth_and_resign((const void *)imp,
                                    ptrauth_key_process_dependent_code,
                                    modifierForSEL(sel, cls),
                                    ptrauth_key_function_pointer, 0);
#elif CACHE_IMP_ENCODING == CACHE_IMP_ENCODING_ISA_XOR
        return (IMP)(imp ^ (uintptr_t)cls);
#elif CACHE_IMP_ENCODING == CACHE_IMP_ENCODING_NONE
        return (IMP)imp;
#else
#error Unknown method cache IMP encoding.
#endif
    }

    template <Atomicity, IMPEncoding>
    void set(SEL newSel, IMP newImp, Class cls);
}
```

**cache**结构体的核心是类型为**bucket_t**的指针，指向以**_imp**和**_sel**对应的缓存点。

> **uintptr_t**数据类型定义为：
> ```objc
#ifndef _UINTPTR_T
#define _UINTPTR_T
typedef unsigned long           uintptr_t;
#endif /* _UINTPTR_T */
```
>
> 本质类型为无符号长整型数据类型，在runtime中，大多数基本数据类型均为无符号长整型，例如**void \***。

上文已经提到，**cache**的存在是为了优化方法的调用，在Runtime中方法的调用流程：

1. 当要调用一个方法时，首先去当前类的**cache**方法缓存中寻找，如果存在，则直接执行；
2. 如果**cache**中不存在，则会去当前类的方法列表中寻找，找到后执行并将该方法按照实现**_imp**和方法签名**_sel**存放在**cache**中，以便下次快速调用。

* **class_data_bits_t bits：**该变量可以说是**Class**的核心成员变量，本质是一个可以被Mask的指针类型。根据不同的Mask，取出不同的值。

```c
struct class_data_bits_t {
    friend objc_class;

    // Values are the FAST_ flags above.
    uintptr_t bits;
public:
    class_rw_t* data() const {
        return (class_rw_t *)(bits & FAST_DATA_MASK);
    }
    void setData(class_rw_t *newData)
    {
        ASSERT(!data()  ||  (newData->flags & (RW_REALIZING | RW_FUTURE)));
        // Set during realization or construction only. No locking needed.
        // Use a store-release fence because there may be concurrent
        // readers of data and data's contents.
        uintptr_t newBits = (bits & ~FAST_DATA_MASK) | (uintptr_t)newData;
        atomic_thread_fence(memory_order_release);
        bits = newBits;
    }
    // 省略其他方法
    ...
}
```

**class_data_bits_t bits**仅仅包含一个成员变量**bits**，该变量不仅包含指针，同时包含**Class**的各种异或flag，当需要取出信息时，需要用对应的**FAST_**前缀开头的flag掩码对**bits**进行按位与操作。

例如，在获取信息的方法

```c
class_rw_t* data() const {
    return (class_rw_t *)(bits & FAST_DATA_MASK);
}
```

中，通过对**bits**进行**FAST_DATA_MASK**的与操作，返回**class_rw_t \***。**class_rw_t**以及**class_ro_t**可以说是**Class**中的核心结构，其定义如下：

```c
struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint16_t witness;

    Class firstSubclass;
    Class nextSiblingClass;

    // 方法列表
    const method_array_t methods();
    // 属性列表
    const property_array_t properties();
    // 协议列表
    const protocol_array_t protocols();

    // 省略其他方法
    ...
}

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
    const uint8_t * ivarLayout;
    
    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    // 省略其他方法
    ...
}
```

在**class_rw_t**结构体中，方法列表**method_array_t**、属性列表**property_array_t**、协议列表**protocol_array_t**是可以被Runtime修改和扩展的。

而在**class_ro_t**结构体中包含了类的名称**name**、方法列表**method_list_t**、属性列表**property_list_t**、协议列表**protocol_list_t**等类的基本信息，**class_ro_t**中的信息时不允许修改，并且不可扩展的。

**objc_class  <-   class_data_bits_t  ->  FAST_DATA_MASK  <-  class_rw_t   <-  class_ro_t**

## realizeClass

```c
static Class realizeClassWithoutSwift(Class cls, Class previously)
{
    // 省略
    ...

    auto ro = (const class_ro_t *)cls->data();
    auto isMeta = ro->flags & RO_META;
    if (ro->flags & RO_FUTURE) {
        // This was a future class. rw data is already allocated.
        rw = cls->data();
        ro = cls->data()->ro();
        ASSERT(!isMeta);
        cls->changeInfo(RW_REALIZED|RW_REALIZING, RW_FUTURE);
    } else {
        // Normal class. Allocate writeable class data.
        rw = objc::zalloc<class_rw_t>();
        rw->set_ro(ro);
        rw->flags = RW_REALIZED|RW_REALIZING|isMeta;
        cls->setData(rw);
    }

    // 省略
    ...
}
```

**realizeClass**是Runtime构造一个完整的类的入口，在没有调用**realizeClass**之前，类是不完整的。上述函数中，最开始返回的仅仅是**class_ro_t**类的基本信息，在进行**realizeClass**时，将类的Category中定义的各种扩展附加到类上，同时改写**data()**的返回值为**class_rw_t**类型。

因此一个类的完整信息保存在结构**class_rw_t**中，**class_ro_t**结构仅仅保存类的基本信息。

## objc_object

上述**objc_class**继承自**objc_object**，也就是说在Objective-C中，类也是一个对象，而对象在runtime中被定义为**objc_object结构体**。

```c
struct objc_object {
private:
    isa_t isa;

public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // rawISA() assumes this is NOT a tagged pointer object or a non pointer ISA
    Class rawISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();
    
    // 省略其他方法
    ...
}
```

**objc_object结构体**的定义相对较为简单，其中仅包含一个**isa_t**的联合体类型。

```c
union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
};
```

**isa_t**是一个联合体，可以表示**Class**或者**uintptr_t**类型，在Objective-C 2.0 中大多数使用的是**uintptr_t**类型。其中**bits**是一个64位的数据，每一位或者几位都表示了关于当前对象的信息。

## 再探objc_object和objc_class

**objc_class**继承自**objc_object**，说明在**objc_class**中也有一个**isa**属性，此时这个属性表示当前类属于哪个类，而这种说明类是属于哪个类的类，称之为**元类（meta-class）**。

元类并不是类的父类，元类在Objective-C的消息转发机制中会由详解，此时，只需要直到，每一个类都有一个与之对应的元类。

## id

在Objective-C 中，**id**类型经常会被使用到，它表示任意类型的类实例变量，在Runtime中，id的定义如下：

```c
typedef struct objc_object *id;
```

其实id类型就是一个**objc_object \***，因为**objc_object**的**isa**的存在，所有Runtime是可以知道id类型对应的真实类型的。

> **Void \***表示任意类型的指针。


## 总结

本文中，从开发者常用的**NSObject**出发，了解了Objective-C语言中类和对象所对应的数据结构**objc_class**和**objc_object**。可以使用一张图了解这三者之间的关系。

![](/assets/runtime/1/nsobject-objc_class-objc_object-relationship.jpg)

