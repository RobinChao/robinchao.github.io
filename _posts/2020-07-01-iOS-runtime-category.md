---
layout: post
author: Robin
title: Runtime剖析04 --- 深入理解Category
tags: Runtime
---

在Objective-C中，可以通过**Category**添加属性、方法、协议，在Runtime中**Class**和**Category**都是通过结构体实现的。和**Category**相似的还有**Extension**，二者的区别在于，**Extension**在编译期就直接和原类编译在一起，而**Category**是在运行时动态添加到原类中的。

## Category的数据结构

在Runtime中，Category的数据结构是基本的结构体，其定义如下：

```c
typedef struct category_t *Category;

struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
    
    protocol_list_t *protocolsForMeta(bool isMeta) {
        if (isMeta) return nullptr;
        else return protocols;
    }
};
```

**Category**的定义中，为每一种可添加的元素提供了属性，添加实例方法的**instanceMethods**、添加类方法的**classMethods**、添加协议的**protocols**、添加实例属性的**instanceProperties**，并且提供了添加方法、属性和协议的可行性判断方法。

## Category的加载

首先，Category数据会被保存在Mach-O中的**__data**段，当Objective-C被dyld加载的时候，Objective-C开始进入其入口函数**_objc_init()**。

```c
void _objc_init(void)
{
    static bool initialized = false;
    if (initialized) return;
    initialized = true;
    
    // fixme defer initialization until an objc-using image is found?
    environ_init();
    tls_init();
    static_init();
    runtime_init();
    exception_init();
    cache_init();
    _imp_implementationWithBlock_init();

    _dyld_objc_notify_register(&map_images, load_images, unmap_image);

#if __OBJC2__
    didCallDyldNotifyRegister = true;
#endif
}
```

在入口函数**_objc_init()**中，会进行各种初始化操作，例如runtime、exception、cache等等的初始化，其中**_dyld_objc_notify_register()**注册函数会向dyld注册监听Mach-O中Objective-C相关的section被载入和载出内存的事件。该函数有三个回调事件：

1. 对应&map_images回调**_dyld_objc_notify_mapped**：当dyld已经将images加载进内存时；
2. 对应load_images回调**_dyld_objc_notify_init**：当dyld初始化image后；
3. 对应unmap_image回调**_dyld_objc_notify_unmapped**：当dyld将image移除内存时。

将Category写入到目标类的方法列表，发生在**_dyld_objc_notify_mapped**，即Mach-O相关的sections都加载到内存之后发生。此时的回调函数为**map_images**，该函数中最终调用的是**_read_images**：

```c
 if (hCount > 0) {
      _read_images(hList, hCount, totalClasses, unoptimizedTotalClasses);
 }

 // objc/Source/objc-os.mm
```

在**_read_images**中，又调取了

```c
if (didInitialAttachCategories) {
    for (EACH_HEADER) {
        load_categories_nolock(hi);
    }
}
 // objc/Source/objc-runtime-new.mm
```

最终，关于Category的读取转向了函数**load_categories_nolock**，在这个函数中，将对Category的属性、方法、协议等进行读取，最终通过**unattachedCategories**方法添加到目标类的方法列表中。

```c
static void load_categories_nolock(header_info *hi) {
    bool hasClassProperties = hi->info()->hasCategoryClassProperties();

    size_t count;
    auto processCatlist = [&](category_t * const *catlist) {
        for (unsigned i = 0; i < count; i++) {
            category_t *cat = catlist[i];
            Class cls = remapClass(cat->cls);
            locstamped_category_t lc{cat, hi};

            if (!cls) {
                // Category's target class is missing (probably weak-linked).
                // Ignore the category.
                if (PrintConnecting) {
                    _objc_inform("CLASS: IGNORING category \?\?\?(%s) %p with "
                                 "missing weak-linked target class",
                                 cat->name, cat);
                }
                continue;
            }

            // Process this category.
            if (cls->isStubClass()) {
                // Stub classes are never realized. Stub classes
                // don't know their metaclass until they're
                // initialized, so we have to add categories with
                // class methods or properties to the stub itself.
                // methodizeClass() will find them and add them to
                // the metaclass as appropriate.
                if (cat->instanceMethods ||
                    cat->protocols ||
                    cat->instanceProperties ||
                    cat->classMethods ||
                    cat->protocols ||
                    (hasClassProperties && cat->_classProperties))
                {
                    objc::unattachedCategories.addForClass(lc, cls);
                }
            } else {
                // First, register the category with its target class.
                // Then, rebuild the class's method lists (etc) if
                // the class is realized.
                if (cat->instanceMethods ||  cat->protocols
                    ||  cat->instanceProperties)
                {
                    if (cls->isRealized()) {
                        attachCategories(cls, &lc, 1, ATTACH_EXISTING);
                    } else {
                        objc::unattachedCategories.addForClass(lc, cls);
                    }
                }

                if (cat->classMethods  ||  cat->protocols
                    ||  (hasClassProperties && cat->_classProperties))
                {
                    if (cls->ISA()->isRealized()) {
                        attachCategories(cls->ISA(), &lc, 1, ATTACH_EXISTING | ATTACH_METACLASS);
                    } else {
                        objc::unattachedCategories.addForClass(lc, cls->ISA());
                    }
                }
            }
        }
    };

    processCatlist(_getObjc2CategoryList(hi, &count));
    processCatlist(_getObjc2CategoryList2(hi, &count));
}
```

1. 首先调用**_getObjc2CategoryList**读取**__objc_catlist** section下所有记录的**category**，并存放在**category_t * const *catlist**数组中；
2. 遍历**category_t * const *catlist**数组；
3. 对每一个**category_t *cat = catlist[i];**，先调用**remapClass**获取**Category**所属的类；
4. 找到Category对应的类对象**cls**后，开始对**cls**进行修改工作。首先，如果category中有实例方法、协议，以及实例属性的话，则直接对**cls**进行操作，如果category中包含了类方法、协议或属性之一的话，还需要对**cls**所对应的**元类（cls->ISA()）**进行操作；
5. 无论是对cls还是cls的元类进行操作，都调用方法**attachCategories**，修改class的方法列表结构。

## attachCategories

```c
static void
attachCategories(Class cls, const locstamped_category_t *cats_list, uint32_t cats_count,
                 int flags)
{
    if (slowpath(PrintReplacedMethods)) {
        printReplacements(cls, cats_list, cats_count);
    }
    if (slowpath(PrintConnecting)) {
        _objc_inform("CLASS: attaching %d categories to%s class '%s'%s",
                     cats_count, (flags & ATTACH_EXISTING) ? " existing" : "",
                     cls->nameForLogging(), (flags & ATTACH_METACLASS) ? " (meta)" : "");
    }

    /*
     * Only a few classes have more than 64 categories during launch.
     * This uses a little stack, and avoids malloc.
     *
     * Categories must be added in the proper order, which is back
     * to front. To do that with the chunking, we iterate cats_list
     * from front to back, build up the local buffers backwards,
     * and call attachLists on the chunks. attachLists prepends the
     * lists, so the final result is in the expected order.
     */
    constexpr uint32_t ATTACH_BUFSIZ = 64;
    method_list_t   *mlists[ATTACH_BUFSIZ];
    property_list_t *proplists[ATTACH_BUFSIZ];
    protocol_list_t *protolists[ATTACH_BUFSIZ];

    uint32_t mcount = 0;
    uint32_t propcount = 0;
    uint32_t protocount = 0;
    bool fromBundle = NO;
    bool isMeta = (flags & ATTACH_METACLASS);
    auto rwe = cls->data()->extAllocIfNeeded();

    for (uint32_t i = 0; i < cats_count; i++) {
        auto& entry = cats_list[i];

        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            if (mcount == ATTACH_BUFSIZ) {
                prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
                rwe->methods.attachLists(mlists, mcount);
                mcount = 0;
            }
            mlists[ATTACH_BUFSIZ - ++mcount] = mlist;
            fromBundle |= entry.hi->isBundle();
        }

        property_list_t *proplist =
            entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            if (propcount == ATTACH_BUFSIZ) {
                rwe->properties.attachLists(proplists, propcount);
                propcount = 0;
            }
            proplists[ATTACH_BUFSIZ - ++propcount] = proplist;
        }

        protocol_list_t *protolist = entry.cat->protocolsForMeta(isMeta);
        if (protolist) {
            if (protocount == ATTACH_BUFSIZ) {
                rwe->protocols.attachLists(protolists, protocount);
                protocount = 0;
            }
            protolists[ATTACH_BUFSIZ - ++protocount] = protolist;
        }
    }

    if (mcount > 0) {
        prepareMethodLists(cls, mlists + ATTACH_BUFSIZ - mcount, mcount, NO, fromBundle);
        rwe->methods.attachLists(mlists + ATTACH_BUFSIZ - mcount, mcount);
        if (flags & ATTACH_EXISTING) flushCaches(cls);
    }

    rwe->properties.attachLists(proplists + ATTACH_BUFSIZ - propcount, propcount);

    rwe->protocols.attachLists(protolists + ATTACH_BUFSIZ - protocount, protocount);
}
```

在该方法中，会完成对category中方法、协议、属性的加载，最终添加到对应class的方法、协议以及属性列表中。最终会调用**attachLists**方法，进行原类和分类列表的合并。

```c
    void attachLists(List* const * addedLists, uint32_t addedCount) {
        if (addedCount == 0) return;

        if (hasArray()) {
            // many lists -> many lists
            uint32_t oldCount = array()->count;
            uint32_t newCount = oldCount + addedCount;
            setArray((array_t *)realloc(array(), array_t::byteSize(newCount)));
            array()->count = newCount;
            memmove(array()->lists + addedCount, array()->lists, 
                    oldCount * sizeof(array()->lists[0]));
            memcpy(array()->lists, addedLists, 
                   addedCount * sizeof(array()->lists[0]));
        }
        else if (!list  &&  addedCount == 1) {
            // 0 lists -> 1 list
            list = addedLists[0];
        } 
        else {
            // 1 list -> many lists
            List* oldList = list;
            uint32_t oldCount = oldList ? 1 : 0;
            uint32_t newCount = oldCount + addedCount;
            setArray((array_t *)malloc(array_t::byteSize(newCount)));
            array()->count = newCount;
            if (oldList) array()->lists[addedCount] = oldList;
            memcpy(array()->lists, addedLists, 
                   addedCount * sizeof(array()->lists[0]));
        }
    }
```


在合并的时候，会采用**头插法**将新的列表插入到原始列表中，也就是新的列表会位于原始列表的头部位置。这也就解释了为什么category中的方法，会`覆盖`class的原始方法。严格说并不是覆盖，而是因为category中定义的方法会排列在方法列表的前面，在进行方法查找的时候，会优先被找到，从而调用，而原始方法位置相对靠后，导致没有查找到，因此没有被调动。

## Category和+load方法

**+load**方法在Objective-C中被调用的时机相对较早，因此Category中定义的方法需要在调用**+load**方法之前就被附加到类的方法列表中，否则可能导致调用无效的情况，实际上，Runtime中也是真么保证的。

```c
void _objc_init(void)
{
    static bool initialized = false;
    if (initialized) return;
    initialized = true;
    
    // fixme defer initialization until an objc-using image is found?
    environ_init();
    tls_init();
    static_init();
    runtime_init();
    exception_init();
    cache_init();
    _imp_implementationWithBlock_init();

    _dyld_objc_notify_register(&map_images, load_images, unmap_image);
    
    didCallDyldNotifyRegister = true;
}
```

在**_dyld_objc_notify_register**中，runtime向dyld中注册了三个事件监听：**map_images、load_images、unmap_image**。在**map_images**事件中，runtime会读取Mach-O文件中的oc sections，并根据这些信息初始化runtime环境，在这一步包含了category的加载等。之后当runtime环境初始化完成后，会进行dyld的**load_images**，这一步会调用**+load**方法。

## Category Associate

在项目开发中，经常会使用到**Category**，有时候会遇到向**Category**中添加属性的需求，但是在**Category**中不能直接添加属性，而是要借助Runtime提供的**Associated**相关的API，例如：

```objectivec
// 声明
@interface NSObject (TestCategory)
@property (nonatomic, strong) NSObject *object;
@end

// 实现
#import <objc/runtime.h>

static void *const kAssociatedObjectKey = (void *)&kAssociatedObjectKey;

@implementation NSObject (TestCategory)


- (NSObject *)object{
    return objc_getAssociatedObject(self, kAssociatedObjectKey);
}

- (void)setObject:(NSObject *)object{
    objc_setAssociatedObject(self, kAssociatedObjectKey, object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end
```

在**Category**中添加属性后，默认是没有实现方法的，如果调用属性会发生崩溃，而且还会提示如下警告。

```c
Property 'object' requires method 'object' to be defined - use @dynamic or provide a method implementation in this category

Property 'object' requires method 'setObject:' to be defined - use @dynamic or prov ide a method implementation in this category
```

这里提出**objc_getAssociatedObject**的runtime源码，如下：

```c
id
_object_get_associative_reference(id object, const void *key)
{
    ObjcAssociation association{};

    {
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.get());
        AssociationsHashMap::iterator i = associations.find((objc_object *)object);
        if (i != associations.end()) {
            ObjectAssociationMap &refs = i->second;
            ObjectAssociationMap::iterator j = refs.find(key);
            if (j != refs.end()) {
                association = j->second;
                association.retainReturnedValue();
            }
        }
    }

    return association.autoreleaseReturnedValue();
}
```

从源码中可以看到，所有通过**Associated**添加的属性，都被存储在一个全局单独的**AssociationsHashMap**哈希表中，**objc_getAssociatedObject**和**objc_setAssociatedObject**本质都是在操作这个哈希表，通过对哈希表进行映射来存储对象。

在**Associated**的API中还有一些内存管理相关的关键字，例如**OBJC_ASSOCIATION_ASSIGN**，这些关键字用来指定对象的内存管理方式，在runtime中会根据这些关键字进行不同的内存管理。

```c
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```

