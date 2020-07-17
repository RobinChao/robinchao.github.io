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

在**isa_t**的定义中，有两个重要的成员变量**cls**和**struct**，并且**struct**部分是在定义了**ISA_BITFIELD**之后才会使用的，也就是说，只有符合ISA_BITFIELD的时候，才会使用**struct**结构体，即采用了优化的isa策略时，**isa_t**类型并不等同于**Class**，而是一个**struct**结构。这个结构共占用了64个字节，从低位的**nonpointer**到高位的**extra_rc**，定义中中**:**表示该成员占用几个字节。

各成员的含义如下表所示：

|成员变量|占用位(单位：bit)|含义|备注|
|-----|-----|-----|-----|
|nonpointer|1|标志位。1 代表开启了isa优化，0 代表未开启isa优化|可根据此位判断对象是否启用了isa优化。|
|has_assoc|1|标志位。标识该对象是否有关联对象。|对象没有关联对象时，其内存释放更快。|
|has_cxx_dtor|1|标志位。标识对象是否有C++或ARC析构函数。|无析构函数时，内存释放更快。|
|shiftcls|33|类指针的非零位。||
|magic|6|“魔法”位。固定值0x1a，用于调试时区分对象是否已经初始化。||
|weakly_referenced|1|标志位。标识对象是否被别的对象弱引用。|没有弱引用的对象内存释放更快。|
|deallocating|1|标志位。标识对象是否正在被释放。||
|has_sidetable_rc|1|标志位。标识对象当前的引用计数是否过大，如果过大则需要使用sidetable存储引用计数。||
|extra_rc|19|记录当前对象的引用计数。||

其中和对象引用计数相关的有**has_sidetable_rc**和**extra_rc**，如上所述，当对象引用计数过大时，**has_sidetable_rc**会被设定为1，并启用sidetable来存储引用计数。

### SideTable

**SideTable**是一个全局的引用计数表，其中存储了项目中所有对象的引用计数。在弄清楚**SideTable**和**extra_rc**之间的关系之前，先了解一下Runtime是如何添加对象的引用计数的。

```c
ALWAYS_INLINE id 
objc_object::rootRetain(bool tryRetain, bool handleOverflow)
{
	// 如果是Tagged Pointer类型，直接返回this。因为TaggedPointer类型不使用引用计数管理内存
    if (isTaggedPointer()) return (id)this;

    bool sideTableLocked = false;
    // 标记 extra_rc 是否溢出，默认false
    bool transcribeToSideTable = false;

    // 临时变量，用于isa_t的存储方式切换
    isa_t oldisa;
    isa_t newisa;

    do {
        transcribeToSideTable = false;
        // 先取出isa_t
        oldisa = LoadExclusive(&isa.bits);
        newisa = oldisa;
        // 如果没有启用isa优化，则返回对应的sidetable记录
        if (slowpath(!newisa.nonpointer)) {
            ClearExclusive(&isa.bits);
            if (rawISA()->isMetaClass()) return (id)this;
            if (!tryRetain && sideTableLocked) sidetable_unlock();
            if (tryRetain) return sidetable_tryRetain() ? (id)this : nil;
            else return sidetable_retain();
        }
        // don't check newisa.fast_rr; we already called any RR overrides
        // 如果对象正在析构，则返回nil
        if (slowpath(tryRetain && newisa.deallocating)) {
            ClearExclusive(&isa.bits);
            if (!tryRetain && sideTableLocked) sidetable_unlock();
            return nil;
        }
        // 是否溢出标记
        uintptr_t carry;
        // 调用 addc 函数 对 extra_rc 执行 ++ 操作，返回 carry
        newisa.bits = addc(newisa.bits, RC_ONE, 0, &carry);  // extra_rc++
        // 如果有溢出，则表明extra_rc已经溢出。
        // 1. 先将 extra_rc 减半
        // 2. 然后将另一半转存至sidetable
        if (slowpath(carry)) {
            // newisa.extra_rc++ overflowed
            // 如果本次不处理溢出，则递归调用一次，并设置handleOverflow为true，下次处理
            if (!handleOverflow) {
                ClearExclusive(&isa.bits);
                return rootRetain_overflow(tryRetain); // return rootRetain(tryRetain, true);
            }
            // Leave half of the retain counts inline and 
            // prepare to copy the other half to the side table.
            // 进行具体的溢出处理：
            // 1. 使用 RC_HALF 宏定义，对 extra_rc 进行减半操作
            // 2. 设置has_sidetable_rc标记为true
            // 3. transcribeToSideTable 标记为true
            if (!tryRetain && !sideTableLocked) sidetable_lock();
            sideTableLocked = true;
            transcribeToSideTable = true;
            newisa.extra_rc = RC_HALF;
            newisa.has_sidetable_rc = true;
        }
    } while (slowpath(!StoreExclusive(&isa.bits, oldisa.bits, newisa.bits)));

    // transcribeToSideTable为true时，将extra_rc减半的一部分，转存到sidetable中
    if (slowpath(transcribeToSideTable)) {
        // Copy the other half of the retain counts to the side table.
        sidetable_addExtraRC_nolock(RC_HALF);
    }

    if (slowpath(!tryRetain && sideTableLocked)) sidetable_unlock();
    return (id)this;
}
```

在Runtime中，通过**SideTable**来管理对象的引用计数和弱引用。**SideTable**中会包含三部分内容：自旋锁、引用计数表和弱引用表。

```c
struct SideTable {
    spinlock_t slock; // 自旋锁
    RefcountMap refcnts; // 引用计数表
    weak_table_t weak_table; // 弱引用表

    SideTable() {
        memset(&weak_table, 0, sizeof(weak_table));
    }

    ~SideTable() {
        _objc_fatal("Do not delete SideTable.");
    }

    void lock() { slock.lock(); }
    void unlock() { slock.unlock(); }
    void forceReset() { slock.forceReset(); }

    // Address-ordered lock discipline for a pair of side tables.

    template<HaveOld, HaveNew>
    static void lockTwo(SideTable *lock1, SideTable *lock2);
    template<HaveOld, HaveNew>
    static void unlockTwo(SideTable *lock1, SideTable *lock2);
};
```

**SideTable**本质上也是一个结构体，多个**SideTable**会构成一个集合，一张**SideTable**会管理多个对象，因此通常被称为**SideTables**，在系统中是全局唯一的。

```c
static objc::ExplicitInit<StripedMap<SideTable>> SideTablesMap;

static StripedMap<SideTable>& SideTables() {
    return SideTablesMap.get();
}
```

**SideTables**被包裹在**StripedMap**类中，每个对象在进行引用计数管理时，都需要通过**StripedMap**的哈希算法，找到对应的**SideTable**表，之后再对引用计数进行管理。

```c
// StripedMap 哈希算法
static unsigned int indexForPointer(const void *p) {
    uintptr_t addr = reinterpret_cast<uintptr_t>(p);
    return ((addr >> 4) ^ (addr >> 9)) % StripeCount;
}
```

从**SideTable**的定义中得知，引用计数表**refcnts**的数据类型为**RefcountMap**，而**RefcountMap**实际上是一个**模板类 DenseMap**。

```c
typedef objc::DenseMap<DisguisedPtr<objc_object>,size_t,RefcountMapValuePurgeable> RefcountMap;
```

简单理解**DenseMap**就是一个**map**，其中**key**是**DisguisedPtr<objc_object>**，**value**是对应的引用计数，另外在该**map**中会检测引用计数，当引用计数为0时，会自动将对象的引用计数数据清空。

```c
void shrink_and_clear() {
    unsigned OldNumEntries = NumEntries;
    this->destroyAll();

    // Reduce the number of buckets.
    unsigned NewNumBuckets = 0;
    if (OldNumEntries)
      NewNumBuckets = std::max(MIN_BUCKETS, 1 << (Log2_32_Ceil(OldNumEntries) + 1));
    if (NewNumBuckets == NumBuckets) {
      this->BaseT::initEmpty();
      return;
    }

    operator delete(Buckets);
    init(NewNumBuckets);
  }
```

在Objective-C中，当要获取一个对象的引用计数时，Runtime会分为三种情况进行获取，分别对应Tagged Pointer、优化的isa和未优化的isa。

```c
inline uintptr_t 
objc_object::rootRetainCount()
{
	// 如果是Tagged Pointer，则直接返回this
    if (isTaggedPointer()) return (uintptr_t)this;

    sidetable_lock();
    isa_t bits = LoadExclusive(&isa.bits);
    ClearExclusive(&isa.bits);
    // 如果是优化的isa_t
    if (bits.nonpointer) {
        uintptr_t rc = 1 + bits.extra_rc;
        // 如果使用了sidetable，则从sidetable中获取引用计数
        if (bits.has_sidetable_rc) {
        	// 总的引用计数 = rc部分 + sidetable部分
            rc += sidetable_getExtraRC_nolock();
        }
        sidetable_unlock();
        return rc;
    }

    sidetable_unlock();
    // 如果是未优化的isa_t，则返回sidetable中的数据
    return sidetable_retainCount();
}
```

在获取优化的isa_t类型对象的引用计数时，注意是要取两部分的记录，然后进行汇总，对应添加引用计数的步骤。在取SideTable部分记录的引用计数时，需要注意在记录中并不是直接获取，而是要根据存储的情况进行获取。

```c
size_t 
objc_object::sidetable_getExtraRC_nolock()
{
    ASSERT(isa.nonpointer);
    SideTable& table = SideTables()[this];
    // 找到对应的引用计数表
    RefcountMap::iterator it = table.refcnts.find(this);
    // 如果表为空，则返回0
    if (it == table.refcnts.end()) return 0;
    // 否则先进行移位操作，然后返回结果
    else return it->second >> SIDE_TABLE_RC_SHIFT;
}
```

**#define SIDE_TABLE_RC_SHIFT 2**宏定义直接使用在引用计数的获取流程中，是因为在引用计数表的低2位的位置存储的并不是引用计数，而是记录当前对象是否有弱引用，以及是否正在deallocing。

### 弱引用表

在**SideTable**的定义中，还有一个非常重要的属性**weak_table_t weak_table**，前文已经了解，**weak_table**是当前对象的弱引用表，存储着弱引用相关的信息，在Runtime中，其定义如下：

```c
struct weak_table_t {
    weak_entry_t *weak_entries; // hash数组，存储弱引用对象的信息
    size_t    num_entries; // hash数组中元素个数
    uintptr_t mask; // hash数组长度-1，参与hash计算 
    uintptr_t max_hash_displacement; // 发生hash冲突的最大次数
};
```

**weak_table_t**中相对重要的是**weak_entry_t**类型的数组部分，可通过hash算法找到对应的对象在数组中的index，另外，**weak_table_t**具有动态扩容的特性，而**sidetables**的大小是固定64个。

**weak_entries**本质上是一个hash数组，数组中存储着**weak_entry_t**类型的元素。**weak_entry_t**定义如下：

```c
/**
 * The internal structure stored in the weak references table. 
 * It maintains and stores
 * a hash set of weak references pointing to an object.
 * If out_of_line_ness != REFERRERS_OUT_OF_LINE then the set
 * is instead a small inline array.
 */
#define WEAK_INLINE_COUNT 4

// out_of_line_ness field overlaps with the low two bits of inline_referrers[1].
// inline_referrers[1] is a DisguisedPtr of a pointer-aligned address.
// The low two bits of a pointer-aligned DisguisedPtr will always be 0b00
// (disguised nil or 0x80..00) or 0b11 (any other address).
// Therefore out_of_line_ness == 0b10 is used to mark the out-of-line state.
#define REFERRERS_OUT_OF_LINE 2

struct weak_entry_t {
    DisguisedPtr<objc_object> referent; // 弱引用对象

    // 联合体，引用该对象的对象列表。
    // 引用个数小于4，使用inline_referrers数组
    // 引用个数大于4，使用weak_referrer_t *referrers动态数组
    union {
        struct {
            weak_referrer_t *referrers; // 弱引用该对象的对象数组，动态
            uintptr_t        out_of_line_ness : 2; // 是否使用动态数组的标记
            uintptr_t        num_refs : PTR_MINUS_2; // 动态数组中的元素个数
            uintptr_t        mask; // 参与hash计算，大小为数组大小-1，最终确定数组index
            uintptr_t        max_hash_displacement; // 最大hash冲突次数
        };
        struct {
            // out_of_line_ness field is low bits of inline_referrers[1]
            weak_referrer_t  inline_referrers[WEAK_INLINE_COUNT];
        };
    };

    bool out_of_line() {
        return (out_of_line_ness == REFERRERS_OUT_OF_LINE);
    }

    weak_entry_t& operator=(const weak_entry_t& other) {
        memcpy(this, &other, sizeof(other));
        return *this;
    }

    weak_entry_t(objc_object *newReferent, objc_object **newReferrer)
        : referent(newReferent)
    {
        inline_referrers[0] = newReferrer;
        for (int i = 1; i < WEAK_INLINE_COUNT; i++) {
            inline_referrers[i] = nil;
        }
    }
};
```

在查找弱引用对象的时候，始终使用的是hash定位的方式，在runtime中，弱引用所使用的hash定位算法如下：

```c
static weak_entry_t *
weak_entry_for_referent(weak_table_t *weak_table, objc_object *referent)
{
    ASSERT(referent);

    // 获取弱引用表中所有弱引用对象
    weak_entry_t *weak_entries = weak_table->weak_entries;

    if (!weak_entries) return nil;
    // 确定hash值对应数组的开始索引，内部调用 ptr_hash 函数
    // 1. 获取到对象的hash指针
    // 2. 与如引用表的 mask 进行 位与 运算
    // 这样的目的是为了减小数值，便于计算。1000...000类型转变为 011...1 的形式
    size_t begin = hash_pointer(referent) & weak_table->mask;
    size_t index = begin;
    size_t hash_displacement = 0;
    // 遍历对象数组
    while (weak_table->weak_entries[index].referent != referent) {
    	// 加入位与运算，防止数组下标越界
        index = (index+1) & weak_table->mask;
        // 寻找一轮后，没有找到对应元素，触发 bad_weak_table
        if (index == begin) bad_weak_table(weak_table->weak_entries);
        // hash冲突增加
        hash_displacement++;
        // 如果hash冲突大于最大可能冲突次数，说明目标元素不在数组中，返回nil
        if (hash_displacement > weak_table->max_hash_displacement) {
            return nil;
        }
    }

    return &weak_table->weak_entries[index];
}

static inline uint32_t ptr_hash(uint64_t key)
{
    key ^= key >> 4;
    key *= 0x8a970be7488fda55;
    key ^= __builtin_bswap64(key);
    return (uint32_t)key;
}
```

在进行hash定位的时候，有一个巧妙的操作语句：

```c
index = (index+1) & weak_table->mask;
```

该语句会在当前位置的下一个相邻位置进行查找，同时当查找到最后一个位置时，会自动从数组的第一个位置开始查找，也就是巧妙的实现了数组的轮转查找，也保证了数组下表不会越界。

由于弱引用表的大小不是固定的，而是随着元素的插入和删除进行动态调整大小的，因此关键在于学习Runtime是如何对其进行动态大小调整的。

#### 动态扩容

扩容主要发生在表格容量满的时候，而进行扩容前，而在Runtime中会进行提前扩容，需要先判断存储表是否需要进行扩容，判断方法如下：

```c
#define TABLE_SIZE(entry) (entry->mask ? entry->mask + 1 : 0)

// Grow the given zone's table of weak references if it is full.
static void weak_grow_maybe(weak_table_t *weak_table)
{
    size_t old_size = TABLE_SIZE(weak_table);

    // Grow if at least 3/4 full.
    if (weak_table->num_entries >= old_size * 3 / 4) {
        weak_resize(weak_table, old_size ? old_size*2 : 64);
    }
}
```

判断的依据在于 **weak_table->num_entries >= old_size * 3 / 4**，即当当前存储表的容量仅剩下1/4的时候，会进行扩容操作。具体的扩容操作是在**weak_resize**中进行的，也就是后文所说的**容量重置**部分。

进行扩容后，存储表的容量会比原有容量大一倍，这么做的目的在于，放置后序频繁的进行内存申请，以及既然这次要扩容，后序扩容的几率会更大，不如一次扩多点。

#### 动态收缩

存储表容量的压缩通常发生在删除了其中一些元素之后，此时系统会调用**weak_compact_maybe**判断当前存储表是否需要收缩：

```c
// Shrink the table if it is mostly empty.
static void weak_compact_maybe(weak_table_t *weak_table)
{
    size_t old_size = TABLE_SIZE(weak_table);

    // Shrink if larger than 1024 buckets and at most 1/16 full.
    if (old_size >= 1024  && old_size / 16 >= weak_table->num_entries) {
        weak_resize(weak_table, old_size / 8);
        // leaves new table no more than 1/2 full
    }
}
```

此时判断的依据是**old_size >= 1024  && old_size / 16 >= weak_table->num_entries**，即容量已经超过1024字节以及存储表容量最多只使用了1/16的时候，会进行容量收缩处理，而收缩是按照现有容量的八倍大小进行收缩的。

#### 容量重置

无论是扩容，还是收缩，都调用了**weak_resize**进行容量的处理。

```c
static void weak_resize(weak_table_t *weak_table, size_t new_size)
{
    size_t old_size = TABLE_SIZE(weak_table);
    // 取出原始数据
    weak_entry_t *old_entries = weak_table->weak_entries;
    // 给新的容量申请内存
    weak_entry_t *new_entries = (weak_entry_t *)
        calloc(new_size, sizeof(weak_entry_t));
    // 重置weak_table容量相关属性
    weak_table->mask = new_size - 1; // 数组大小-1
    weak_table->weak_entries = new_entries; // 元素
    weak_table->max_hash_displacement = 0; // 最大hash冲突重置
    weak_table->num_entries = 0;  // restored by weak_entry_insert below
    
    if (old_entries) {
        weak_entry_t *entry;
        weak_entry_t *end = old_entries + old_size;
        // 遍历元素数组，将元素重新插入到新的存储表中
        for (entry = old_entries; entry < end; entry++) {
            if (entry->referent) {
                weak_entry_insert(weak_table, entry);
            }
        }
        // 最后释放掉老的内存空间
        free(old_entries);
    }
}

// 元素的插入操作
static void weak_entry_insert(weak_table_t *weak_table, weak_entry_t *new_entry)
{
    weak_entry_t *weak_entries = weak_table->weak_entries;
    ASSERT(weak_entries != nil);

    size_t begin = hash_pointer(new_entry->referent) & (weak_table->mask);
    size_t index = begin;
    size_t hash_displacement = 0;
    while (weak_entries[index].referent != nil) {
        index = (index+1) & weak_table->mask;
        if (index == begin) bad_weak_table(weak_entries);
        hash_displacement++;
    }

    weak_entries[index] = *new_entry;
    weak_table->num_entries++;

    if (hash_displacement > weak_table->max_hash_displacement) {
        weak_table->max_hash_displacement = hash_displacement;
    }
}
```

也就是说，弱引用存储表的容量在扩容的时候，并不是在原有存储表上进行直接扩容的，而是根据新的大小开辟了一块新的内存空间，同时将老的数据完整迁移到新的内存空间上，然后释放掉老的存储表。

## autoreleasepool

在iOS中，第三种内存管理的方式是**autoreleasepool**，在ARC中，通常直接使用**@autoreleasepool{}**的方式使用，其中包裹的对象的内存管理工作就交给了自动释放池进行管理。

关于**autoreleasepool**这里不再详述，具体可查看开源项目[objc](https://github.com/opensource-apple/objc4)。