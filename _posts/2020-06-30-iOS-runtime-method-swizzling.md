---
layout: post
author: Robin
title: Runtime剖析03 --- “黑魔法” Method Swizzling 
tags: Runtime
---

方法替换，又称为**Method Swizzling**，是Objective-C语言中比较流行的“黑魔法”。动态替换方法实现，大多数情况下使用在一些检测类的业务逻辑中，同时，方法替换也带给开发者更多可能的新的开发方式。在简单剖析**Method Swizzling **前，先看看方法替换场景中两种经常遇到的情况。

1. 需要替换的方法在目标类中有实现；
2. 需要替换的方法在目标类中没有实现，但再其父类中有实现。

对于第一种情况，直接可以使用runtime提供的**method_exchangeImplementations**即可。

```c
// 方法定义
OBJC_EXPORT void
method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2) 
    OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);

// 方法实现
void method_exchangeImplementations(Method m1, Method m2)
{
    if (!m1  ||  !m2) return;
    // 加锁
    mutex_locker_t lock(runtimeLock);
    // 方法交换
    IMP m1_imp = m1->imp;
    m1->imp = m2->imp;
    m2->imp = m1_imp;

    // 缓存清理
    flushCaches(nil);
    // 设定标识
    adjustCustomFlagsForMethodChange(nil, m1);
    adjustCustomFlagsForMethodChange(nil, m2);
}
```

对于第二种情况，稍微复杂一点。由于在目标类中并没有待替换方法的实现，而再其父类中有实现，那么其实要交换的是目标类父类中的实现，但是这样一替换后，其他调用父类这个方法的地方，也会被转发到所替换的方法中，这样明显是不合理的。

为了避免这种情况，在进行方法替换前，首先要检查目标类中是否有对应方法的实现，如果没有，则要将方法动态添加进当前类的方法列表中。

```objectivec
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // When swizzling a Instance method, use the following:
        Class class = [self class];
        
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        
        SEL originalSelector = @selector(systemMethod_PrintLog);
        SEL swizzledSelector = @selector(swizzledMethod_PrintLog);
        
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
        BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));
        
        if (didAddMethod) {
            class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        }else{
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}
```

* 直接使用**class_addMethod()**检查目标class中是否有方法实现，如果目标类中没有对应方法的实现，**didAddMethod**会返回**true**，同时会将**originalSelector**添加到目标类中，如果目标类中已经有了方法的实现，**didAddMethod**会返回**false**，那么直接调用**method_exchangeImplementations()**交换即可；
* 在进行**class_addMethod()**的时候，**SEL**传入的是待替换的**originalSelector**，但是**IMP**传入的是**swizzledMethod**的实现，在完成**class_addMethod()**后，即完成了方法实现的替换，即使用了替换方法的实现，替换了原始方法的实现。
* 当**class_addMethod()**完成后，然后调用**class_replaceMethod()**对**swizzledSelector**的实现进行替换。在**class_replaceMethod()**的内部实现中，首先会尝试调用**class_addMethod()**，将方法添加到类中，如果添加失败，则说明class中已经有了该方法，此时会调用**method_setImplementation**设置方法的**IMP**。


另外，**class**的获取，如果替换的是实例方法，则直接使用**Class class = [self class];**获取实例对象，如果替换的是类方法，则需要使用**Class class = object_getClass((id)self);**获取类对象。

方法替换的时机一般情况下在**+ (void)load **方法中进行，因为该方法在runtime将class加载进内存时，其加载时机比较靠前，能够保证方法替换成功后，调用的逻辑符合预期。

## Method swizzling 如何工作的

**号称Objective-C中的黑魔法 --- Method swizzling，其本质是基于runtime底层数据结构体的应用。**  因此了解runtime底层数据结构有助于理解该黑魔法的工作原理等。

### class & object_getClass

首先从上文中的**class**对象的获取说起：

```objectivec
// When swizzling a Instance method, use the following:
Class class = [self class];
        
// When swizzling a class method, use the following:
Class class = object_getClass((id)self);
```

**class**

在**NSObject**的定义中，**class**的定义有两个，一个是实例方法，一个是类方法。

```objectivec
+ (Class)class {
    return self;
}

- (Class)class {
    return object_getClass(self);
}
```

当调用者是类对象时，返回的是类对象本身，而调用者是实例对象时，会调用runtime的**object_getClass**方法，该方法中具体做了什么呢？

**object_getClass**

```c
Class object_getClass(id obj)
{
    if (obj) return obj->getIsa();
    else return Nil;
}
```

**object_getClass**的内部实现非常简单，就是获取对象的**isa**指针。如果对象时**实例对象**，**isa**返回的是实例对象所对应的类对象；如果是**类对象**，**isa**返回的是类对象对应的**元类对象**。

> **方法替换中，如果替换的是实例方法，则需要修改实例对象所对应的类对象的方法列表，如果是类方法，则需要修改类对象所对应的元类对象的方法列表。**

## class_getInstanceMethod

在确认了目标类之后，接下来就是要准备方法替换的原始方法和替换方法：**originalMethod**、**swizzledMethod**。**Method**数据类型的定义如下：

```c
typedef struct method_t *Method;

struct method_t {
    SEL name;  // 方法名
    const char *types; // 方法返回值类型和参数类型，TypeEncoding
    MethodListIMP imp; // 方法实现

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

在方法列表中，存储的既是**method_t**结构体类型。通过**class_getInstanceMethod**取出方法，既是通过**SEL**在指定类对象的方法列表中查找对应的**Method**。

```c
Method class_getInstanceMethod(Class cls, SEL sel)
{
    if (!cls  ||  !sel) return nil;

    // This deliberately avoids +initialize because it historically did so.

    // This implementation is a bit weird because it's the only place that 
    // wants a Method instead of an IMP.
        
    // Search method lists, try method resolver, etc.
    lookUpImpOrForward(nil, sel, cls, LOOKUP_RESOLVER);
    return _class_getMethod(cls, sel);
}
```

**class_getInstanceMethod**中调用了**lookUpImpOrForward**进行**imp**的搜索以及方法缓存构建，之后会调用**_class_getMethod**方法，根据目标**cls**和**sel**查找对应的**Method**。

```c
static Method _class_getMethod(Class cls, SEL sel)
{
    mutex_locker_t lock(runtimeLock);
    return getMethod_nolock(cls, sel);
}

static method_t *
getMethod_nolock(Class cls, SEL sel)
{
    method_t *m = nil;

    runtimeLock.assertLocked();

    // fixme nil cls?
    // fixme nil sel?

    ASSERT(cls->isRealized());

    while (cls  &&  ((m = getMethodNoSuper_nolock(cls, sel))) == nil) {
        cls = cls->superclass;
    }

    return m;
}

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

**_class_getMethod**方法直接调用**getMethod_nolock**方法，在该方法中验证类的继承链，向上查找SEL对应对应的method，其搜索条件是通过cls和sel在cls的方法列表中，查找对应的method。

## class_addMethod

在获取到目标类和方法之后，首先尝试的是调用**class_addMethod**将**swizzledMethod**添加到目标类的方法列表中。

目的在于，如果目标类中没有要替换的**originalMethod**，则直接将**swizzledMethod**作为**originalMethod**的实现添加到目标类中，如果目标类中存在**originalMethod**的实现，则**class_addMethod**方法会添加失败，返回**false**，此时调用**method_exchangeImplementations**直接替换**originalMethod**的实现为**swizzledMethod**的实现即可。

**class_addMethod**方法的实现如下：

```c
BOOL 
class_addMethod(Class cls, SEL name, IMP imp, const char *types)
{
    if (!cls) return NO;

    mutex_locker_t lock(runtimeLock);
    return ! addMethod(cls, name, imp, types ?: "", NO);
}

static IMP 
addMethod(Class cls, SEL name, IMP imp, const char *types, bool replace)
{
    IMP result = nil;

    runtimeLock.assertLocked();

    checkIsKnownClass(cls);
    
    ASSERT(types);
    ASSERT(cls->isRealized());

    method_t *m;
    if ((m = getMethodNoSuper_nolock(cls, name))) {
        // already exists
        if (!replace) {
            result = m->imp;
        } else {
            result = _method_setImplementation(cls, m, imp);
        }
    } else {
        auto rwe = cls->data()->extAllocIfNeeded();

        // fixme optimize
        method_list_t *newlist;
        newlist = (method_list_t *)calloc(sizeof(*newlist), 1);
        newlist->entsizeAndFlags = 
            (uint32_t)sizeof(method_t) | fixed_up_method_list;
        newlist->count = 1;
        newlist->first.name = name;
        newlist->first.types = strdupIfMutable(types);
        newlist->first.imp = imp;

        prepareMethodLists(cls, &newlist, 1, NO, NO);
        rwe->methods.attachLists(&newlist, 1);
        flushCaches(cls);

        result = nil;
    }

    return result;
}
```

**class_addMethod**方法中调用**addMethod**方法，并设定参数**replace**为**NO**，即不进行替换，仅仅添加，这也揭示了为什么当存在method的实现时，该方法会返回false。

**addMethod**方法中，首先会检查锁提供的method是否在cls的方法列表中存在，如果存在则直接获取，如果不存在，则会根据现有方法列表的数据重新创建一个方法列表对象**method_list_t**，并将提供的method追加到该方法列表中，最后刷新设定类的方法列表，完成方法的添加。

## class_replaceMethod

如果上一步**class_addMethod**返回成功，则说明在目标类中添加SEL为**originalMethod**，IMP为**swizzledMethod**的方法，那么接下来就剩下对方法的实现进行替换了，此时调用**class_replaceMethod**。

**class_replaceMethod**方法中还是调用**addMethod**方法，但是参数**replace**设定为**YES**，表示执行替换的逻辑。并且此时，在方法列表中是确定存在对应的方法的，因此会直接调用**_method_setImplementation**方法，设定方法实现。

## method_exchangeImplementations

如果**class_addMethod**返回失败，说明目标类中的**originalMethod**已经存在，此时直接对其实现进行交换即可，交换方法的实现调用**method_exchangeImplementations**。

```c
void method_exchangeImplementations(Method m1, Method m2)
{
    if (!m1  ||  !m2) return;

    mutex_locker_t lock(runtimeLock);

    IMP m1_imp = m1->imp;
    m1->imp = m2->imp;
    m2->imp = m1_imp;


    // RR/AWZ updates are slow because class is unknown
    // Cache updates are slow because class is unknown
    // fixme build list of classes whose Methods are known externally?

    flushCaches(nil);

    adjustCustomFlagsForMethodChange(nil, m1);
    adjustCustomFlagsForMethodChange(nil, m2);
}
```

**method_exchangeImplementations**方法的核心其实就是交换两个方法的实现imp。

## 注意点

在使用Method Swizzlings的时候，有几个要注意的地方：

**1. 加载时机**

swizzling应该只在**+load**中完成。 

在 Objective-C 的运行时中，每个类有两个方法都会自动调用。+load 是在一个类被初始装载时调用，+initialize 是在应用第一次调用该类的类方法或实例方法前调用的。两个方法都是可选的，并且只有在方法被实现的情况下才会被调用。

**2. 单例**

swizzling 应该只在 dispatch_once 中完成, 由于 swizzling 改变了全局的状态，所以我们需要确保每个预防措施在运行时都是可用的。原子操作就是这样一个用于确保代码只会被执行一次的预防措施，就算是在不同的线程中也能确保代码只执行一次s。Grand Central Dispatch 的 dispatch_once 满足了所需要的需求，并且应该被当做使用 swizzling 的初始化单例方法的标准。

**3. _cmd调用**

通常在swizzling交换后的方法中，还需要再调用一次本方法，这样做并不会产生递归调用，因为此时调用的已经是交换后的方法，而再次调用的目的是触发交换的方法实现执行。