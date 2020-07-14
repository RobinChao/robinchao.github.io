---
layout: post
author: Robin
title: Runtime剖析05 --- 再议iOS内存管理
tags: Runtime
---

我们都知道，iOS中进行内存管理的管理模型是**引用计数**，但是这属于上层应用的范畴，在系统底层，iOS会根据不同的数据结构或者不同的数据类型，进行系统内存的分区，在不同的分区中，管理着自己的内存，另外，iOS的内存管理并不直接管理硬件内存，而是管理着硬件内存之上的一个过渡内存---**虚拟内存**。关于虚拟内存，可参考[iOS虚拟内存管理](https://robinchao.github.io/iOS-VMManage/)一文。

## iOS 内存分区

iOS的内存管理是基于虚拟内存的管理，虚拟内存能够让每一个进程都在逻辑上`独占`整个设备的内存。iOS又将虚拟内存按照地址由低到高划分为五大区：

![](/assets/runtime/5/vm-zone)

虚拟内中，最上方是系统内核区的内存，最下方是系统保留的内存空间，中间则是程序加载的内存空间。内存按照自下而上，由低地址到高地址的拓展，程序加载到内存分为三段：

1. 未初始化数据(.bss)：存放未进行初始化的静态变量、全局变量
2. 已初始化数据(.data)：存放已初始化的静态变量、全局变量
3. 代码段(.text)：存放代码的二进制代码

其他内存段**栈区**和**堆区**，分别用于方法或函数的调用和开发者创建的对象等内存。也就是说，开发者所管理的内存是在堆区，堆地址的分配不连续，但是整体地址是由低到高拓展。栈区的内存管理是由系统自动管理，栈区的地址是连续的，其内存地址是由高向低拓展。在程序运行时，栈区和堆区的大小是变化的，只不过栈区是由系统管理的，堆区是通过引用计数的方式管理对象的，内存的管理也是由开发者管理的。

## Tagged Pointer

在开发的过程中，难免会有些数字需要进行存储，而在iOS中，通常使用NSNumber对象来表示数字，对于绝大多数程序而言，所使用到的数字并不会很大，也用不到上亿的数字，同样对于字符串类型，绝大多数情况下，字符的个数也在8个字节以内。在iPhone 5s之后，iOS的寻址地址扩大到了64位，可以使用63位来表示一个数字，一位用来作为符号位。此时如果存储一个数字，例如**NSNumber *num=@10000**，远远达不到63位的内存，这样在内存中则会留下很多无用的空位，造成内存空间的浪费。

针对上述问题，Apple引入了**Tagged Pointer**，一种特殊的**指针**，在该类型的指针中，存储的已经不是地址，而是**真实的数据和一些附加信息**。

**此部分内容不再详述，具体可查看[深入理解 Tagged Pointer
](https://www.infoq.cn/article/deep-understanding-of-tagged-pointer/)。**

在Runtime中，针对Tagged Pointer的辨别，首先需要一个标志位，用来判断当前指针是**真正的指针**还是**Tagged Pointer**，Runtime中使用了一个宏定义**define _OBJC_TAG_MASK (1UL<<63)**，表示如果64位数据中，最高位是1的话，则表明当前是一个**Tagged Pointer**类型。在Runtime中，不仅仅NSNumber有Tagged Pointer类型，还有NSString、NSIndexPath、NSDate等，具体如下定义：

```c
{
    // 60-bit payloads
    OBJC_TAG_NSAtom            = 0, 
    OBJC_TAG_1                 = 1, 
    OBJC_TAG_NSString          = 2, 
    OBJC_TAG_NSNumber          = 3, 
    OBJC_TAG_NSIndexPath       = 4, 
    OBJC_TAG_NSManagedObjectID = 5, 
    OBJC_TAG_NSDate            = 6,

    // 60-bit reserved
    OBJC_TAG_RESERVED_7        = 7, 

    // 52-bit payloads
    OBJC_TAG_Photos_1          = 8,
    OBJC_TAG_Photos_2          = 9,
    OBJC_TAG_Photos_3          = 10,
    OBJC_TAG_Photos_4          = 11,
    OBJC_TAG_XPC_1             = 12,
    OBJC_TAG_XPC_2             = 13,
    OBJC_TAG_XPC_3             = 14,
    OBJC_TAG_XPC_4             = 15,
    OBJC_TAG_NSColor           = 16,
    OBJC_TAG_UIColor           = 17,
    OBJC_TAG_CGColor           = 18,
    OBJC_TAG_NSIndexSet        = 19,

    OBJC_TAG_First60BitPayload = 0, 
    OBJC_TAG_Last60BitPayload  = 6, 
    OBJC_TAG_First52BitPayload = 8, 
    OBJC_TAG_Last52BitPayload  = 263, 

    OBJC_TAG_RESERVED_264      = 264
};
```

例如**0xa**转换为二进制，得到**1010**，其中高位**1xxx**表明是一个**Tagged Pointer**,而剩下的3位**010**,表示是一个**NSString**类型，即**010**转换为十进制为**2**，对应上述定义中的**OBJC_TAG_NSString = 2**。

对于字符串来说，只有小字符串会被存储为**Tagged Pointer**类型，那么到底要多小呢？能够想到的是，字符串在进行春初的时候，并不是存储着字符串本身，而是字符串中每个字符的ASCII码，在字符串长度增加到8个字符之前，字符串是按照小对象的方式存储的，更大的字符串则是使用传统的指针方式存储的。

```objectivec
    NSString *test_small = @"a";
    NSString *str = [NSString stringWithFormat:@"%@", test_small];
    
    NSNumber *number = @1.0;
    NSNumber *number_large = @3.1415926;
```

![](/assets/runtime/5/tagged-pointer.jpg)

**isa**

从上述示例的结果中可以看到，当一个对象被存储未**Tagged Pointer**类型后，该对象的**isa**指针是**0x0**，指向空的，也就是说**Tagged Pointer**类型的对象，是没有isa属性的。在Runtime中，获取一个对象的isa指针定义如下：

```c
inline Class 
objc_object::getIsa() 
{
    if (fastpath(!isTaggedPointer())) return ISA();

    extern objc_class OBJC_CLASS_$___NSUnrecognizedTaggedPointer;
    uintptr_t slot, ptr = (uintptr_t)this;
    Class cls;

    slot = (ptr >> _OBJC_TAG_SLOT_SHIFT) & _OBJC_TAG_SLOT_MASK;
    cls = objc_tag_classes[slot];
    if (slowpath(cls == (Class)&OBJC_CLASS_$___NSUnrecognizedTaggedPointer)) {
        slot = (ptr >> _OBJC_TAG_EXT_SLOT_SHIFT) & _OBJC_TAG_EXT_SLOT_MASK;
        cls = objc_tag_ext_classes[slot];
    }
    return cls;
}

static inline bool 
_objc_isTaggedPointer(const void * _Nullable ptr)
{
    return ((uintptr_t)ptr & _OBJC_TAG_MASK) == _OBJC_TAG_MASK;
}
```

当获取一个对象的isa指针式，如果是tagged pointer类型，则会取出高4位的内容，进行对象类型的确定。

## NONPOINTER_ISA

**NONPOINTER_ISA**是iOS中另一种内存管理的方式，即对象的isa指针，该指针用来表明对象属性和类类型。Apple同样优化了该中方式的内存管理方式，在isa中，不仅表明了属性属于那个类，还附加了引用计数**extra_rc**、是否weak引用**weakly_referenced**、是否有附加属性**has_assoc**等附加信息。

```c
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}

typedef struct objc_class *Class;

struct objc_class : objc_object {
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
}
 
struct objc_object {
private:
    isa_t isa;
}

union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    struct {
        ISA_BITFIELD;  // defined in isa.h
    };
#endif

# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
    struct {
        uintptr_t nonpointer        : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33; // MACH_VM_MAX_ADDRESS 0x1000000000
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;
#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };
}
```

**isa指针**其本质是**isa_t 联合类型**。联合类型的作用在于用更少的空间，表示更多可能的类型，但是类型之间是不能共存的。
