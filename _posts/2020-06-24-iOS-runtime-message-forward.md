---
layout: post
author: Robin
title: Runtime剖析02 --- 消息与消息发送机制
tags: Runtime
---

在Objective-C中，消息发送指Runtime会根据**SEL**查找对应的**IMP**，当查找到，则调用函数指针进行方法调用，若查找不到，则进入动态消息解析和消息转发流程，如果动态解析和消息转发失败，则程序会崩溃。

## 消息相关数据结构

### SEL

**SEL**称之为消息选择器，相当于一个**key**，在类的消息列表中，可以根据这个**key**查找对应的消息实现**IMP**。

在Runtime中，SEL的定义如下：

```c
/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;
```

可以看到**SEL**其实是一个**objc_selector \***结构体指针，但是在苹果开源的runtime中并没有其定义，目前**SEL**仅是一个字符串。

虽然**SEL**可以作为key对消息进行查找，但是当不同的类有着相同的**SEL**的时候，再进行消息实现查找时，可能无法确定消息实现真实的归属，因此在进行消息实现查找时，会结合消息发送的目标Class，才能找到具体的最终的**IMP**。

### method_t

开篇已说，runtime会根据**SEL**查找对应的实现**IMP**。具体地说，runtime会在Class的方法列表中查找方法的实现，在方法列表中方法的实现是以**method_t**结构体的形式存储的。

```c
struct method_t {
    SEL name;
    const char *types;
    MethodListIMP imp;

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

**method_t**结构体重包含了**SEL**的名称，以及指向对应试下的**imp**指针，另外**types**指的是方法的返回值和参数类型，其格式一般为**v24@0:8@16**，此种格式被称为**Type Encodings**，对应的解释说明详见[Type Encodings](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)。

### IMP

**IMP**本质上是一个函数指针，用于指向方法的具体实现，在runtime中，其定义如下：

```c
/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
#endif
```

> **IMP**是由编译器生成的，如果知道了**IMP**的地址，就可以绕过runtime的消息发送过程，直接调用函数实现。

在消息发送过程中，runtime会根据**id**和**SEL**来唯一确定**IMP**并进行调用。

## 消息

在Objective-C中，函数的调用被称为**消息发送**。在进行代码编译时，代码会被修改为**objc_msgSend**的格式。

```c
OBJC_EXPORT id _Nullable
objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
    OBJC_AVAILABLE(10.0, 2.0, 9.0, 1.0, 2.0);
```

**objc_msgSend()**形式即为Objective-C中的消息发送的入口，该函数的具体实现是由汇编语言实现的，其目的一是为了提高执行效率，二是因为该函数的返回值类型是可变的，汇编正好具有返回值类型多样性的特性。

除了**objc_msgSend()**之外，编译器还会根据具体的情况，将消息转发改写为如下形式之一：

* objc_msgSend
* objc_msgSend_stret
* objc_msgSendSuper
* objc_msgSendSuper_stret

当消息发送给当前类的Super Class的时候，编译器会将消息发送改写为**objc_msgSendSuper**的格式。

```c
OBJC_EXPORT id _Nullable
objc_msgSendSuper(struct objc_super * _Nonnull super, SEL _Nonnull op, ...)
    OBJC_AVAILABLE(10.0, 2.0, 9.0, 1.0, 2.0);
```

在**objc_msgSendSuper**函数的参数中，第一个参数不再是当前类的指针，而变为**objc_super \***结构体指针，**objc_super**结构体包含两个数据，**receiver**指调用super方法的对象，即消息接收者，而**super_class**表示当前子类的父类对象。

```c
struct objc_super {
    /// Specifies an instance of a class.
    __unsafe_unretained _Nonnull id receiver;
    /// Specifies the particular superclass of the instance to message. 
    __unsafe_unretained _Nonnull Class super_class;
};
```

> 带有**_stret**的函数，表示方法返回的是结构体类型。


## objc_msgSend

```armasm
	ENTRY _objc_msgSend
	UNWIND _objc_msgSend, NoFrame

	cmp	p0, #0			// nil check and tagged pointer check
#if SUPPORT_TAGGED_POINTERS
	b.le	LNilOrTagged		//  (MSB tagged pointer looks negative)
#else
	b.eq	LReturnZero
#endif
	ldr	p13, [x0]		// p13 = isa
	GetClassFromIsa_p16 p13		// p16 = class
LGetIsaDone:
	// calls imp or objc_msgSend_uncached
	CacheLookup NORMAL, _objc_msgSend

#if SUPPORT_TAGGED_POINTERS
LNilOrTagged:
	b.eq	LReturnZero		// nil check
// 省略其他
``` 

1. 进入**objc_msgSend**后，首先通过**cmp	p0, #0**检查函数参数**receiver**是否为**nil**，如果为**nil**，则进入**LReturnZero**，返回0；
2. 如果不为**nil**，则将**receiver**的**isa**存储在**p13**寄存器；
3. 在寄存器**p13**中，取出**isa**对应的**Class**，存储到**p16**寄存器；
4. **Class**获取完成后，调用**CacheLookup NORMAL**函数，查找**Class**的方法缓存列表，如果命中，则调用**_objc_msgSend**，如果未命中，则调用**objc_msgSend_uncached**。

**objc_msgSend_uncached**也是汇编语言实现，作用是为方法缓存列表中未查找到方法缓存时，在**Class**的方法列表中进行查找。

```c
STATIC_ENTRY __objc_msgSend_uncached
	UNWIND __objc_msgSend_uncached, FrameWithNoSaves

	// THIS IS NOT A CALLABLE C FUNCTION
	// Out-of-band p16 is the class to search
	
	MethodTableLookup
	TailCallFunctionPointer x17

	END_ENTRY __objc_msgSend_uncached
```

**objc_msgSend_uncached**内部调用了**MethodTableLookup**，**MethodTableLookup**是一个汇编实现的宏定义，其内部调用了C语言函数**lookUpImpOrForward**。

## lookUpImpOrForward

```c
extern IMP lookUpImpOrForward(id obj, SEL, Class cls, int behavior);
```

**lookUpImpOrForward**函数的目的是根据**Class**和**SEL**，在当前类或者当前类的父类中找到方法对应的**IMP**，同时，缓存找到的对应**IMP**到当前类的方法缓存列表中。如果没有找到对应的**IMP**，则会进入到消息转发流程。

```c
IMP lookUpImpOrForward(id inst, SEL sel, Class cls, int behavior)
{
    const IMP forward_imp = (IMP)_objc_msgForward_impcache;
    IMP imp = nil;
    Class curClass;

    runtimeLock.assertUnlocked();

    // Optimistic cache lookup
    // 首先在根据class和sel在方法缓存中查找imp
    if (fastpath(behavior & LOOKUP_CACHE)) {
        imp = cache_getImp(cls, sel);
        // 如果查找到，则直接进入到done_nolock
        if (imp) goto done_nolock;
    }

    // runtimeLock is held during isRealized and isInitialized checking
    // to prevent races against concurrent realization.

    // runtimeLock is held during method search to make
    // method-lookup + cache-fill atomic with respect to method addition.
    // Otherwise, a category could be added but ignored indefinitely because
    // the cache was re-filled with the old value after the cache flush on
    // behalf of the category.

    runtimeLock.lock();

    // We don't want people to be able to craft a binary blob that looks like
    // a class but really isn't one and do a CFI attack.
    //
    // To make these harder we want to make sure this is a class that was
    // either built into the binary or legitimately registered through
    // objc_duplicateClass, objc_initializeClassPair or objc_allocateClassPair.
    //
    // TODO: this check is quite costly during process startup.
    checkIsKnownClass(cls);

    // 如果class还未realize，先进行realize
    if (slowpath(!cls->isRealized())) {
        cls = realizeClassMaybeSwiftAndLeaveLocked(cls, runtimeLock);
        // runtimeLock may have been dropped but is now locked again
    }
	// 如果class还未initialize，先进行initialize
    if (slowpath((behavior & LOOKUP_INITIALIZE) && !cls->isInitialized())) {
        cls = initializeAndLeaveLocked(cls, inst, runtimeLock);
        // runtimeLock may have been dropped but is now locked again

        // If sel == initialize, class_initialize will send +initialize and 
        // then the messenger will send +initialize again after this 
        // procedure finishes. Of course, if this is not being called 
        // from the messenger then it won't happen. 2778172
    }

    runtimeLock.assertLocked();
    curClass = cls;

    // The code used to lookpu the class's cache again right after
    // we take the lock but for the vast majority of the cases
    // evidence shows this is a miss most of the time, hence a time loss.
    //
    // The only codepath calling into this without having performed some
    // kind of cache lookup is class_getInstanceMethod().
    // 在当前class中没有找到imp，则依次向上查找super class的方法列表
    for (unsigned attempts = unreasonableClassCount();;) {
        // curClass method list.
        // 首先获取当前类的方法体
        Method meth = getMethodNoSuper_nolock(curClass, sel);
        if (meth) {
            imp = meth->imp;
            goto done;
        }
        // 通过继承链，向上查找IMP
        if (slowpath((curClass = curClass->superclass) == nil)) {
            // No implementation found, and method resolver didn't help.
            // Use forwarding.
            imp = forward_imp;
            break;
        }

        // Halt if there is a cycle in the superclass chain.
        if (slowpath(--attempts == 0)) {
            _objc_fatal("Memory corruption in class list.");
        }

        // Superclass cache.
        // 父类缓存
        imp = cache_getImp(curClass, sel);
        if (slowpath(imp == forward_imp)) {
            // Found a forward:: entry in a superclass.
            // Stop searching, but don't cache yet; call method
            // resolver for this class first.
            break;
        }
        // 找到父类的IMP，并缓存
        if (fastpath(imp)) {
            // Found the method in a superclass. Cache it in this class.
            goto done;
        }
    }

    // No implementation found. Try method resolver once.
    // 没有查找到IMP，进入动态方法解析流程
    if (slowpath(behavior & LOOKUP_RESOLVER)) {
        behavior ^= LOOKUP_RESOLVER;
        return resolveMethod_locked(inst, sel, cls, behavior);
    }

 done:
 	// 记录并缓存IMP
    log_and_fill_cache(cls, imp, sel, inst, curClass);
    runtimeLock.unlock();
 done_nolock:
 	// 未找到对应的IMP，返回nil
    if (slowpath((behavior & LOOKUP_NIL) && imp == forward_imp)) {
        return nil;
    }
    return imp;
}
```

**lookUpImpOrForward**工作流程：

1. 尝试在当前receiver对应class的cache中查找imp，如果查找到，则调用；
2. 如果在cache中为查找到imp，则在class的方法列表中查找imp；
3. 尝试在class的所有super class中查找imp。（首先在super class的cache中查找，如果为找到，则在super class的方法列表中查找）；
4. 如果还未找到imp，则尝试进行动态方法解析SEL；
5. 动态解析失败，则尝试进入消息转发流程，让其他class处理SEL。

在查找class的方法列表中是否有SEL对应的IMP时，调用的是**getMethodNoSuper_nolock()**：

```c
static method_t *
getMethodNoSuper_nolock(Class cls, SEL sel)
{
    runtimeLock.assertLocked();

    ASSERT(cls->isRealized());
    // fixme nil cls? 
    // fixme nil sel?

    auto const methods = cls->data()->methods();
    for (auto mlists = methods.beginLists(),
              end = methods.endLists();
         mlists != end;
         ++mlists)
    {
        // <rdar://problem/46904873> getMethodNoSuper_nolock is the hottest
        // caller of search_method_list, inlining it turns
        // getMethodNoSuper_nolock into a frame-less function and eliminates
        // any store from this codepath.
        method_t *m = search_method_list_inline(*mlists, sel);
        if (m) return m;
    }

    return nil;
}
```

首先取出class的方法列表**method_array_t**，然后调用**search_method_list_inline()**，根据SEL查找对应的**method_t**。

```c
ALWAYS_INLINE static method_t *
search_method_list_inline(const method_list_t *mlist, SEL sel)
{
    int methodListIsFixedUp = mlist->isFixedUp();
    int methodListHasExpectedSize = mlist->entsize() == sizeof(method_t);
    
    if (fastpath(methodListIsFixedUp && methodListHasExpectedSize)) {
        return findMethodInSortedMethodList(sel, mlist);
    } else {
        // Linear search of unsorted method list
        for (auto& meth : *mlist) {
            if (meth.name == sel) return &meth;
        }
    }
    return nil;
}
```

在方法查找的时候，会分为两种方式：

1. 如果方法列表是有序的，则使用**findMethodInSortedMethodList**进行前向查找，使用**二分查找**方式
2. 否则直接进行遍历

## objc_msgSendSuper

上文已经提到，当消息发送给当前类的Super Class的时候，编译器会将消息发送改写为**objc_msgSendSuper**的格式。

```c
OBJC_EXPORT id _Nullable
objc_msgSendSuper(struct objc_super * _Nonnull super, SEL _Nonnull op, ...)
    OBJC_AVAILABLE(10.0, 2.0, 9.0, 1.0, 2.0);
```

在实际开发中，当使用**super**关键字调用方法时，编译器则会将代码编译为**objc_msgSendSuper**的格式。

**super关键字本质上类似一个“语法糖”，在代码编译时，编译器会将其替换为objc_super指针类型，来传入到objc_msgSendSuper方法中，而并不是父类的意思。**

```c
struct objc_super {
    /// Specifies an instance of a class.
    // 消息的接收者，一般为当前类的实例对象。
    __unsafe_unretained _Nonnull id receiver;

    /// Specifies the particular superclass of the instance to message. 
    // 告知查找方法IMP的去向，当前类实例的父类对象。
    __unsafe_unretained _Nonnull Class super_class;
};
```

**当使用super关键字调用方法时，runtime会到当前类的父类中查找对应IMP，然后将消息发送到当前类的实例上。**

这也解释了为什么**[self class] 和 [super class]**会输出同样结果的原因。

同样**objc_msgSendSuper**也是由汇编语言实现的，实现如下：

```c
	ENTRY _objc_msgSendSuper
	
	ldr	r9, [r0, #CLASS]	// r9 = struct super->class
	CacheLookup NORMAL, _objc_msgSendSuper
	// cache hit, IMP in r12, eq already set for nonstret forwarding
	ldr	r0, [r0, #RECEIVER]	// load real receiver
	bx	r12			// call imp

	CacheLookup2 NORMAL, _objc_msgSendSuper
	// cache miss
	ldr	r9, [r0, #CLASS]	// r9 = struct super->class
	ldr	r0, [r0, #RECEIVER]	// load real receiver
	b	__objc_msgSend_uncached
	
	END_ENTRY _objc_msgSendSuper
```

寄存器**r9**中保存的是当前实例的父类对象，获取到对应的父类后，调用**CacheLookup**在方法缓存中查找对应imp，缓存命中后，取出**receiver**，调用imp。如果未在缓存中命中IMP，则调用**__objc_msgSend_uncached**，传入**super class**进行方法查找。

## 动态解析

如果在类的继承链中没有找到对应的IMP，runtime则会进入消息的动态解析流程，即进入到**lookUpImpOrForward**中的**resolveMethod_locked**函数调用中。

```c
	if (slowpath(behavior & LOOKUP_RESOLVER)) {
        behavior ^= LOOKUP_RESOLVER;
        return resolveMethod_locked(inst, sel, cls, behavior);
    }
```

**动态解析，就是将方法实现在运行时动态的添加到当前类中。**之后runtime会重新尝试消息查找。

```c
static NEVER_INLINE IMP
resolveMethod_locked(id inst, SEL sel, Class cls, int behavior)
{
    runtimeLock.assertLocked();
    ASSERT(cls->isRealized());

    runtimeLock.unlock();

    if (! cls->isMetaClass()) {
        // try [cls resolveInstanceMethod:sel]
        resolveInstanceMethod(inst, sel, cls);
    } 
    else {
        // try [nonMetaClass resolveClassMethod:sel]
        // and [cls resolveInstanceMethod:sel]
        resolveClassMethod(inst, sel, cls);
        if (!lookUpImpOrNil(inst, sel, cls)) {
            resolveInstanceMethod(inst, sel, cls);
        }
    }

    // chances are that calling the resolver have populated the cache
    // so attempt using it
    return lookUpImpOrForward(inst, sel, cls, behavior | LOOKUP_CACHE);
}
```

在**resolveMethod_locked**中，runtime会根据调用的是实例方法还是类方法，进入到不同的处理逻辑中。

* 动态解析实例方法： **resolveInstanceMethod()**用来动态解析实例方法，在运行时可以动态的将对应的方法实现添加到类实例所对应的类的消息列表中。
* 动态解析类方法： **resolveClassMethod()**用来动态解析类方法，同样可以在运行时动态的将对应的类方法添加到类的消息列表中。

**举例：**

```objectivec
+ (BOOL)resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(test)) {
        class_addMethod([self class], sel, class_getMethodImplementation([self class], @selector(testInstanceMethod)), "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}

- (void)testInstanceMethod{
    NSLog(@"%s", __func__);
}

+ (BOOL)resolveClassMethod:(SEL)sel{
    if (sel == @selector(test)) {
        class_addMethod(object_getClass(self), sel, class_getMethodImplementation(object_getClass(self), @selector(testClassMehotd)), "v@:");
        return YES;
    }
    return [super resolveClassMethod:sel];
}

+ (void)testClassMehotd{
    NSLog(@"%s", __func__);
}
```

在示例中，**test()**方法仅仅声明，没有实现。在运行时，runtime则会进入到消息的动态解析。需要注意的是，动态解析类方法时，方法**class_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types)**中的第一个参数，需要使用**object_getClass()**进行获取后传参。因为在动态解析类方法时，需要将方法的实现添加到当前类的isa指向类中，而类的指向类为**元类**。

* 当**self**是实例对象时，**[self class]** 和 **object_getClass(self)**等价，因为前者会直接调用后者，都是返回对象实例所对应的类。
* 当**self**是类对象时，**[self class]**返回类对象本身，而**object_getClass(self)**返回类对应的元类。

## 消息转发

当动态解析依然失败，runtime则进入到消息转发流程。**消息转发，是将当前消息转发到其他对象进行处理。** 在NSObject中，针对消息转发提供了专门的API来处理。

```objectivec
// 转发类方法，id返回的是类对象
+ (id)forwardingTargetForSelector:(SEL)sel;
// 转发实例方法，id返回的是实例对象
- (id)forwardingTargetForSelector:(SEL)sel;
```

**消息转发示例**

```objectivec
- (id)forwardingTargetForSelector:(SEL)aSelector{
    if (aSelector == @selector(test)) {
        return testForward;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

其中**testForward**为实现了**test**方法的实例对象。如果没有实现**forwardingTargetForSelector**，或者该方法返回**nil**或者**self**，则runtime会进入到另一个转发流程。此时runtime会依次调用**- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector**，获取方法签名，然后根据方法签名，包装成了一个**NSInvocation**对象，并调用**- (void)forwardInvocation:(NSInvocation *)anInvocation**，此时无论转发的消息是否实现，系统都会默认消息已经得到了解析，从而避免崩溃。

![](/assets/runtime/2/message-forward.jpg)

消息转发实际上是将消息转发给另一个对象进行处理，而消息动态解析是在当前类的范围内进行处理。

## 消息转发与多继承

通过消息转发流程，可以模拟实现Objective-C语言的多继承机制，具体可查看[Runtime官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW1)。

## 总结

在Objective-C语言中，方法调用实现的底层机制为**消息发送机制**。在开发中，类的实例对象不能调用类方法，原因是类的实例对象在查找消息IMP的流程仅仅是查找类的方法列表，而对于类方法而言，其实现存放在元类的方法列表中，因此实例对象通过objc_msgSend方法是找不到对应类消息的IMP的。

类大多数情况下是不能调用实例方法的，除非实例方法定义在根类中，即NSObject中。因为当调用类方法是，会在元类的继承链的方法列表中查找对应的IMP，而跟元类对应的父类是NSObject，因此在NSObject中定义的实例方法，其实是可以通过类方法形式来调用的。