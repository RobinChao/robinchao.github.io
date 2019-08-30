---
layout: post
title:  iOS虚拟内存管理
author: Robin
categories: 开发知识
tags: 开发知识 iOS
cover: "/assets/vm/memory_manage.jpg"
---

虚拟内存是一种允许操作系统避开物理RAM限制的内存管理机制。虚拟内存管理器为每个进程创建一个逻辑地址空间或者虚拟内存地址空间，并且将它分配为相同大小的内存块，可称为页。处理器与内存管理单元MMU维持一个页表来映射程序逻辑地址空间到计算机RAM的硬件地址。当程序的代码访问内存中的一个地址时，MMU利用页表将指定的逻辑地址转换为真实的硬件内存地址，这种转换自动发生并且对于运行的应用是透明的。


## 虚拟内存概述

就程序而言，在它逻辑地址空间的地址永远可用。然而，当应用访问一个当前并没有在物理RAM中的内存页的地址时，就会发生页错误。当这种情况发生时，虚拟内存系统调用一个专用的页错误处理器来立即响应错误。页错误处理器停止当前执行的代码，定位到物理内存的一个空闲页，从磁盘加载包含必要数据的页，更新页表，之后返回对程序代码的控制，程序代码就可以正常访问内存地址了。这个过程被称为分页。

如果在物理内存中没有空闲页，页错误处理器必须首先释放一个已经存在的页从而为新页提供空间。由系统平台决定系统如何释放页。在OS X，虚拟内存系统常常将页写入备份存储。备份存储是一个基于磁盘的仓库,包含了给定进程内存页的拷贝。将数据从物理内存移到备份存储被称为页面换出；将数据从备份存储移到物理内存被称为页面换入。在iOS，没有备份存储，所以页面永远不会换出到磁盘，但是只读页仍可以根据需要从磁盘换入。

在OS X and iOS中，页大小为4kb。因此，每次页错误发生时，系统会从磁盘读取4kb。当系统花费过度的时间处理页错误并且读写页，而并不是执行代码时，会发生磁盘震荡（disk thrashing）。

无论页换出／换入，磁盘震荡会降低性能。因为它强迫系统花费大量时间进行磁盘的读写。从备份存储读取页花费相当长的时间，并且比直接从RAM读取要慢很多。如果系统从磁盘读取另一个页之前，不得不将一个页写入磁盘时，性能影响会更糟。

## 虚拟内存的限制

在iOS开发的过程中，难免手动去申请内存，目前大多数的移动设备都是ARM64的设备，即使用的是64位寻址空间，而且在iOS上通过malloc申请的内存只是虚拟内存，不是真正的物理内存，那么在iOS设备上为什么会出现申请了2-3G就会出现申请失败呢？

```c
void *buffer = malloc(2000 * 1024 * 1024);
```

```c
malloc: *** mach_vm_map(size=2097152000) failed (error code=3)
*** error: can't allocate region
```

当申请分配一个超大的内存时，iOS系统会按照`nano_zone`和`scalable_zone`的设计理念进行内存的申请，申请原理如下：

```c
void *    szone_malloc_should_clear(szone_t *szone, size_t size, boolean_t cleared_requested)
{
    void *ptr;
    msize_t msize;

    if (size <= SMALL_THRESHOLD) {
        // tiny size: <1024 bytes (64-bit), <512 bytes (32-bit)
        // think tiny
        msize = TINY_MSIZE_FOR_BYTES(size + TINY_QUANTUM - 1);
        if (!msize) {
            msize = 1;
        }
        ptr = tiny_malloc_should_clear(szone, msize, cleared_requested);
    } else if (size <= szone->large_threshold) {
        // small size: <15k (<1GB machines), <127k (>1GB machines)
        // think small
        msize = SMALL_MSIZE_FOR_BYTES(size + SMALL_QUANTUM - 1);
        if (!msize) {
            msize = 1;
        }
        ptr = small_malloc_should_clear(szone, msize, cleared_requested);
    } else {
        // large: all other allocations
        size_t num_kernel_pages = round_page_quanta(size) >> vm_page_quanta_shift;
        if (num_kernel_pages == 0) { /* Overflowed */
            ptr = 0;
        } else {
            ptr = large_malloc(szone, num_kernel_pages, 0, cleared_requested);
        }
    }
#if DEBUG_MALLOC
    if (LOG(szone, ptr)) {
        malloc_printf("szone_malloc returned %p\n", ptr);
    }
#endif
    /*
     * If requested, scribble on allocated memory.
     */
    if ((szone->debug_flags & MALLOC_DO_SCRIBBLE) && ptr && !cleared_requested && size) {
        memset(ptr, SCRIBBLE_BYTE, szone_size(szone, ptr));
    }

    return ptr;
}
```

* 小于1k的走 tiny_malloc
* 小于15k或者127k的走 small_malloc （视具体不同的设备内存上限不同）
* 剩下的走 large_malloc 

由于我们分配的非常大，我们可以确定我们的逻辑是落入`large_malloc`中。需要特别注意的是： `large_malloc`分配内存的基本单位是一页大小，而对于其他的几种分配方式，则不是必须按照页大小进行分配。

由于 `large_malloc` 这个函数本身并没有特殊需要注意的地方，我们直接关注其真正分配内存的地方，即 `allocate_pages` ，如下所示：

```c
vm_addr = vm_page_quanta_size;
kr = mach_vm_map(mach_task_self(), &vm_addr, allocation_size, allocation_mask, alloc_flags, MEMORY_OBJECT_NULL, 0, FALSE,
            VM_PROT_DEFAULT, VM_PROT_ALL, VM_INHERIT_DEFAULT);
if (kr) {
    szone_error(szone, 0, "can't allocate region", NULL, "*** mach_vm_map(size=%lu) failed (error code=%d)\n", size, kr);
    return NULL;
}
addr = (uintptr_t)vm_addr;
```

从上不难看出，如果分配失败，就是提示报错。而 `mach_vm_map` 则是整个内存的分配核心。

![](/assets/vm/virtual.png)

概括来说，`vm_map` 代表就是一个进程运行时候涉及的虚拟内存， `pmap` 代表的就是和具体硬件架构相关的物理内存。（这里我们暂时先不考虑 `submap` 这种情况）。

`vm_map`本身是进程（或者从Mach内核的角度看是task的地址分布图）。这个地址分布图维护着一个 **双向列表** ，列表的每一项都是 `vm_entry_t` ，代表着虚拟地址上连续的一个范围。而 `pmap` 这个结构体代表了个硬件相关的内存转换：即利用 `pmap` 这个结构体来描述抽象的物理地址访问和使用。

## 进程（任务）的创建

对于在iOS上的进程创建和加载执行Mach-O过程，有必要进行一个简单的介绍，在类UNIX系统本质上是不会无缘无故创建出一个进程的，基本上必须通过`fork`的形式来创建。无论是用户态调用`posix`相关的API还是别的API，最终落入内核是均是通过函数`fork_create_child`来创建属于Mach内核的任务。实现如下：

```c
thread_t
fork_create_child(task_t parent_task, proc_t child, int inherit_memory, int is64bit)
{
	thread_t	child_thread = NULL;
	task_t		child_task;
	kern_return_t	result;

	/* Create a new task for the child process */
	result = task_create_internal(parent_task,
					inherit_memory,
					is64bit,
					&child_task);
	if (result != KERN_SUCCESS) {
		printf("execve: task_create_internal failed.  Code: %d\n", result);
		goto bad;
	}

	/* Set the child task to the new task */
	child->task = child_task;

	/* Set child task proc to child proc */
	set_bsdtask_info(child_task, child);

	/* Propagate CPU limit timer from parent */
	if (timerisset(&child->p_rlim_cpu))
		task_vtimer_set(child_task, TASK_VTIMER_RLIM);

	/* Set/clear 64 bit vm_map flag */
	if (is64bit)
		vm_map_set_64bit(get_task_map(child_task));
	else
		vm_map_set_32bit(get_task_map(child_task));

#if CONFIG_MACF
	/* Update task for MAC framework */
	/* valid to use p_ucred as child is still not running ... */
	mac_task_label_update_cred(child->p_ucred, child_task);
#endif

	/* Set child scheduler priority if nice value inherited from parent */
	if (child->p_nice != 0)
		resetpriority(child);

	/* Create a new thread for the child process */
	result = thread_create(child_task, &child_thread);
	if (result != KERN_SUCCESS) {
		printf("execve: thread_create failed. Code: %d\n", result);
		task_deallocate(child_task);
		child_task = NULL;
	}
bad:
	thread_yield_internal(1);

	return(child_thread);
}
```

* 要注意的就是**Mach内核里面没有进程的概念，只有任务**，进程是属于BSD之上的抽象。它们之间的联系就是通过指针建立， `child_proc->task = child_task`。

`fork`出来的进程像是一个空壳，需要利用这个进程壳去执行科执行文件编程有意义的**程序进程**。从XNU上看，可执行文件的类型有如下分类：

```c
{ exec_mach_imgact,        "Mach-o Binary" },
{ exec_fat_imgact,        "Fat Binary" },
{ exec_shell_imgact,    "Interpreter Script" }
```

常用的通常是`Mach-o`文件：

```c
exec_mach_imgact(struct image_params *imgp)
{
    ... 省略无数

    if ((mach_header->magic == MH_CIGAM) ||
        (mach_header->magic == MH_CIGAM_64)) {
        error = EBADARCH;
        goto bad;
    }

    if ((mach_header->magic != MH_MAGIC) &&
        (mach_header->magic != MH_MAGIC_64)) {
        error = -1;
        goto bad;
    }

    if (mach_header->filetype != MH_EXECUTE) {
        error = -1;
        goto bad;
    }

    if (imgp->ip_origcputype != 0) {
        /* Fat header previously had an idea about this thin file */
        if (imgp->ip_origcputype != mach_header->cputype ||
            imgp->ip_origcpusubtype != mach_header->cpusubtype) {
            error = EBADARCH;
            goto bad;
        }
    } else {
        imgp->ip_origcputype = mach_header->cputype;
        imgp->ip_origcpusubtype = mach_header->cpusubtype;
    }

    task = current_task();
    thread = current_thread();
    uthread = get_bsdthread_info(thread);

    if ((mach_header->cputype & CPU_ARCH_ABI64) == CPU_ARCH_ABI64)
        imgp->ip_flags |= IMGPF_IS_64BIT;

    /* If posix_spawn binprefs exist, respect those prefs. */
    psa = (struct _posix_spawnattr *) imgp->ip_px_sa;
    if (psa != NULL && psa->psa_binprefs[0] != 0) {
        int pr = 0;
        for (pr = 0; pr < NBINPREFS; pr++) {
            cpu_type_t pref = psa->psa_binprefs[pr];
            if (pref == 0) {
                /* No suitable arch in the pref list */
                error = EBADARCH;
                goto bad;
            }

            if (pref == CPU_TYPE_ANY) {
                /* Jump to regular grading */
                goto grade;
            }

            if (pref == imgp->ip_origcputype) {
                /* We have a match! */
                goto grade;
            }
        }
        error = EBADARCH;
        goto bad;
    }
grade:
    if (!grade_binary(imgp->ip_origcputype, imgp->ip_origcpusubtype & ~CPU_SUBTYPE_MASK)) {
        error = EBADARCH;
        goto bad;
    }

    /* Copy in arguments/environment from the old process */
    error = exec_extract_strings(imgp);
    if (error)
        goto bad;

    AUDIT_ARG(argv, imgp->ip_startargv, imgp->ip_argc, 
        imgp->ip_endargv - imgp->ip_startargv);
    AUDIT_ARG(envv, imgp->ip_endargv, imgp->ip_envc,
        imgp->ip_endenvv - imgp->ip_endargv);

    /* reset local idea of thread, uthread, task */
    thread = imgp->ip_new_thread;
    uthread = get_bsdthread_info(thread);
    task = new_task = get_threadtask(thread);

    // 注意点：
    lret = load_machfile(imgp, mach_header, thread, &map, &load_result);

    ... 省略无数
```

上面的代码基本上都是在对文件进行各种检查，然后分配一个预使用的进程壳，之后使用`load_machfile`加载真正的二进制文件。

```c
load_return_t
load_machfile(
    struct image_params    *imgp,
    struct mach_header    *header,
    thread_t         thread,
    vm_map_t         *mapp,
    load_result_t        *result
)
{
    ... 省略一大堆

    if (macho_size > file_size) {
        return(LOAD_BADMACHO);
    }

    result->is64bit = ((imgp->ip_flags & IMGPF_IS_64BIT) == IMGPF_IS_64BIT);

    task_t ledger_task;
    if (imgp->ip_new_thread) {
        ledger_task = get_threadtask(imgp->ip_new_thread);
    } else {
        ledger_task = task;
    }

    // 注意点1
    pmap = pmap_create(get_task_ledger(ledger_task),
               (vm_map_size_t) 0,
               result->is64bit);

    // 注意点2
    map = vm_map_create(pmap,
,
            vm_compute_max_offset(result->is64bit),
            TRUE);

#if defined(__arm64__)
    // 注意点三
    if (result->is64bit) {
        /* enforce 16KB alignment of VM map entries */
        vm_map_set_page_shift(map, SIXTEENK_PAGE_SHIFT);
    } else {
        vm_map_set_page_shift(map, page_shift_user32);
    }
```

* 利用 `pmap_create` 创建硬件相关的物理内存抽象。
* 利用 `vmap_create` 创建虚拟内存的地址图。
* ARM64下的页是16k一个虚拟页对应一个物理页。

这里需要重点关注 `vm_map_create 0` 和 `vm_compute_max_offset(result->is64bit)`，代表着当前任务分配的虚拟内存地址的上下限， `vm_compute_max_offset`函数实现如下：

```c
vm_map_offset_t
vm_compute_max_offset(boolean_t is64)
{
#if defined(__arm__) || defined(__arm64__)
    return (pmap_max_offset(is64, ARM_PMAP_MAX_OFFSET_DEVICE));
#else
    return (is64 ? (vm_map_offset_t)MACH_VM_MAX_ADDRESS : (vm_map_offset_t)VM_MAX_ADDRESS);
#endif
}
```

`pmap_max_offset`函数实现如下：

```c
vm_map_offset_t pmap_max_offset(
    boolean_t    is64 __unused,
    unsigned int    option)
{
    vm_map_offset_t    max_offset_ret = 0;

#if defined(__arm64__)
    assert (is64);
    vm_map_offset_t min_max_offset = SHARED_REGION_BASE_ARM64 + SHARED_REGION_SIZE_ARM64 + 0x20000000; // end of shared region + 512MB for various purposes
    if (option == ARM_PMAP_MAX_OFFSET_DEFAULT) {
        max_offset_ret = arm64_pmap_max_offset_default;
    } else if (option == ARM_PMAP_MAX_OFFSET_MIN) {
        max_offset_ret = min_max_offset;
    } else if (option == ARM_PMAP_MAX_OFFSET_MAX) {
        max_offset_ret = MACH_VM_MAX_ADDRESS;
    } else if (option == ARM_PMAP_MAX_OFFSET_DEVICE) {
        if (arm64_pmap_max_offset_default) {
            max_offset_ret = arm64_pmap_max_offset_default;
        } else if (max_mem > 0xC0000000) {
            max_offset_ret = 0x0000000318000000ULL;     // Max offset is 12.375GB for devices with > 3GB of memory
        } else if (max_mem > 0x40000000) {
            max_offset_ret = 0x0000000218000000ULL;     // Max offset is 8.375GB for devices with > 1GB and <= 3GB of memory
        } else {
            max_offset_ret = min_max_offset;
        }
    } else if (option == ARM_PMAP_MAX_OFFSET_JUMBO) {
        max_offset_ret = 0x0000000518000000ULL;     // Max offset is 20.375GB for pmaps with special "jumbo" blessing
    } else {
        panic("pmap_max_offset illegal option 0x%x\n", option);
    }

    assert(max_offset_ret >= min_max_offset);
    return max_offset_ret;
```

这里的关键点代码是：

```c
if (max_mem > 0xC0000000) {
    max_offset_ret = 0x0000000318000000ULL;     // Max offset is 12.375GB for devices with > 3GB of memory
} else if (max_mem > 0x40000000) {
    max_offset_ret = 0x0000000218000000ULL;     // Max offset is 8.375GB for devices with > 1GB and <= 3GB of memory
} else {
    max_offset_ret = min_max_offset;
}
```

`max_offset_ret` 这个值就代表了我们任务对应的 `vm_map_t` 的最大地址范围，比如说这里是8.375GB。

## 虚拟内存分配的限制

之前提到了 `large_malloc` 会走入到最后的 `vm_map_enter` ，那么我们来看看 `vm_map_enter` 的实现：

```c
vm_map_enter(
    vm_map_t        map,
    vm_map_offset_t        *address,    /* IN/OUT */
    vm_map_size_t        size,
    vm_map_offset_t        mask,
    int            flags,
    vm_map_kernel_flags_t    vmk_flags,
    vm_tag_t        alias,
    vm_object_t        object,
    vm_object_offset_t    offset,
    boolean_t        needs_copy,
    vm_prot_t        cur_protection,
    vm_prot_t        max_protection,
    vm_inherit_t        inheritance)
{

#if CONFIG_EMBEDDED
    // 注意点1:检查页的权限
    if (cur_protection & VM_PROT_WRITE){
        if ((cur_protection & VM_PROT_EXECUTE) && !entry_for_jit){
            printf("EMBEDDED: %s: curprot cannot be write+execute. "
                   "turning off execute\n",
                   __FUNCTION__);
            cur_protection &= ~VM_PROT_EXECUTE;
        }
    }
#endif /* CONFIG_EMBEDDED */

    if (resilient_codesign || resilient_media) {
        if ((cur_protection & (VM_PROT_WRITE | VM_PROT_EXECUTE)) ||
            (max_protection & (VM_PROT_WRITE | VM_PROT_EXECUTE))) {
            return KERN_PROTECTION_FAILURE;
        }
    }

    // 1. 获取任务的可用的地址最小值和最大值
    effective_min_offset = map->min_offset;
    effective_max_offset = map->max_offset;

    if (map->pmap == kernel_pmap) {
        user_alias = VM_KERN_MEMORY_NONE;
    } else {
        user_alias = alias;
    }

#define    RETURN(value)    { result = value; goto BailOut; }

    assert(page_aligned(*address));
    assert(page_aligned(size));

    if (!VM_MAP_PAGE_ALIGNED(size, VM_MAP_PAGE_MASK(map))) {
        clear_map_aligned = TRUE;
    }

StartAgain: ;

    start = *address;

    if (anywhere) {
        vm_map_lock(map);
        map_locked = TRUE;

        if (start < effective_min_offset)
            start = effective_min_offset;
        if (start > effective_max_offset)
            RETURN(KERN_NO_SPACE);


        if( FALSE ) {

        } else {

            if (map->holelistenabled) {
                hole_entry = (vm_map_entry_t)map->holes_list;

                if (hole_entry == NULL) {
                    /*
                     * No more space in the map?
                     */
                    result = KERN_NO_SPACE;
                    goto BailOut;
                } else {

                    boolean_t found_hole = FALSE;

                    do {
                        if (hole_entry->vme_start >= start) {
                            start = hole_entry->vme_start;
                            found_hole = TRUE;
                            break;
                        }

                        if (hole_entry->vme_end > start) {
                            found_hole = TRUE;
                            break;
                        }
                        hole_entry = hole_entry->vme_next;

                    } while (hole_entry != (vm_map_entry_t) map->holes_list);

                    if (found_hole == FALSE) {
                        result = KERN_NO_SPACE;
                        goto BailOut;
                    }

                    entry = hole_entry;

                    if (start == 0)
                        start += PAGE_SIZE_64;
                }
            }
        }

        while (TRUE) {
            vm_map_entry_t    next;

            end = ((start + mask) & ~mask);
            end = vm_map_round_page(end,
                        VM_MAP_PAGE_MASK(map));

            if (end < start)
                RETURN(KERN_NO_SPACE);

            start = end;
            end += size;

            if ((end > effective_max_offset) || (end < start)) {
                RETURN(KERN_NO_SPACE);
            }

            next = entry->vme_next;

            if (map->holelistenabled) {
                if (entry->vme_end >= end)
                    break;
            } else {

                if (next == vm_map_to_entry(map))
                    break;

                if (next->vme_start >= end)
                    break;
            }

            entry = next;

            if (map->holelistenabled) {
                if (entry == (vm_map_entry_t) map->holes_list) {
                    result = KERN_NO_SPACE;
                    goto BailOut;
                }
                start = entry->vme_start;
            } else {
                start = entry->vme_end;
            }

            start = vm_map_round_page(start,
                          VM_MAP_PAGE_MASK(map));
        }

        if (map->holelistenabled) {
            if (vm_map_lookup_entry(map, entry->vme_start, &entry)) {
                panic("Found an existing entry (%p) instead of potential hole at address: 0x%llx.\n", entry, (unsigned long long)entry->vme_start);
            }
        }

        *address = start;
    } 
```

* 注意点1：基本上就是检查页的权限等，iOS上不允许可写和可执行并存。
* 剩下的就是作各种前置检查。

如果上述代码不够清晰明了，如下这段代码可以更加的简洁：

```c
entry = map->first_free;

if (entry == vm_map_to_entry(map)) {
    entry = NULL;
} else {
       if (entry->vme_next == vm_map_to_entry(map)){
            entry = NULL;
       } else {
            if (start < (entry->vme_next)->vme_start ) {
                start = entry->vme_end;
                start = vm_map_round_page(start,
                              VM_MAP_PAGE_MASK(map));
            } else {
                entry = NULL;
            }
       }
}

if (entry == NULL) {
    vm_map_entry_t    tmp_entry;
    if (vm_map_lookup_entry(map, start, &tmp_entry)) {
        assert(!entry_for_jit);
        start = tmp_entry->vme_end;
        start = vm_map_round_page(start,
                      VM_MAP_PAGE_MASK(map));
    }
    entry = tmp_entry;
}
```

* 整个这段代码的意思是，就是要我们要找个一个比我们这个 `start` 地址大的 `vm_entry_t` 。最终的目的是为了在两个已经存在 `vm_entry_t` 之间尝试插入一个能包含从 `start` 到 `start + size` 的新的 `vm_entry_t`。
* 如果没找到的话，就尝试利用 `vm_map_lookup_entry` 找一个 `preceding` 我们地址的的 `vm_entry_t` 。

当找到了一个满足 `start` 地址条件的 `vm_entry_t` 后，剩下就是要满足分配大小 size 的需求了。

```c
while (TRUE) {
    register vm_map_entry_t    next;

    end = ((start + mask) & ~mask);
    end = vm_map_round_page(end,
                VM_MAP_PAGE_MASK(map));
    if (end < start)
        RETURN(KERN_NO_SPACE);

    start = end;
    end += size;

    if ((end > effective_max_offset) || (end < start)) {
        RETURN(KERN_NO_SPACE);
    }

    next = entry->vme_next;

    // 如果是空的头
    if (next == vm_map_to_entry(map))
        break;

    // 如果下一个的start 
    if (next->vme_start >= end)
        break;

    entry = next;
    start = entry->vme_end;
    start = vm_map_round_page(start,
                  VM_MAP_PAGE_MASK(map));
}
*address = start;
assert(VM_MAP_PAGE_ALIGNED(*address,
               VM_MAP_PAGE_MASK(map)));
```

* 判断 `start + size` 是不是可以正好插入在 `vm_entry_t` 代表的地址范围的空隙内，如果一直遍历到最后的任务地址上限都找不到，那就说明不存在我们需求的连续的虚拟内存空间用于作分配了。

## 总结

除了本文说明的虚拟内存分配的连续性限制以外，虚拟内存作为堆内存分配的一种，在布局范围上也有限制。更多详细的信息可参考如下链接。

* [XNU](https://github.com/opensource-apple/xnu)
* [Memory management](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/MemoryManagement.html)
* [iOS内存管理](https://www.jianshu.com/p/4f49c5c81021)
* [理解 iOS 的内存管理](https://blog.devtang.com/2016/07/30/ios-memory-management/)

