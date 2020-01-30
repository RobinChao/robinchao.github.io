---
layout: post
author: Robin
title: \#17\ 排序算法O(n^2)
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/17/cover.jpg'
---

**O(n^2)**的时间复杂度并不是一个最佳的性能表现，但是在某些情况下，该类别的排序算法非常有用，此类算法的空间复杂度表现良好，仅仅需要O(1)的恒定的额外内存空间，对于小型数据集来说，此类排序算法比更为复杂的排序算法更为有利。

在本文中，将学习饿三种不同的、O(n^2)时间复杂度的排序算法：

* 冒泡排序
* 选择排序
* 插入排序

这些算法均是**基于比较**的方法，他们依赖比较的运算，比如小于或等于运算符等，对数据进行排序。比较操作的调用次数是衡量此类算法技术总体性能的一般方法。

## 冒泡排序（Bubble Sort）

冒泡排序是相对简单的一种排序方法，在排序的过程中，重复比较两个数据的大小，并进行数据交换。其中较大的数值会类似气泡上升一样上升到集合的尾部。

### 示例

![](/assets/Data-Structures-&-Algorithms-in-Swift/17/example.png)

例如上图所示的四张扑克，其顺序为*[9， 4， 10， 3]*，现在需要对其从小到大进行排序，需要如下几个步骤：

* 从集合最前端开始，比较扑克牌9和扑克牌4，由于9比4大，因此需要进行位置交换，交换后，顺序变为*[4， 9， 10， 3]*；
* 完成了第一步后，比较的锚点移动到集合的下一个索引处，即此时的扑克牌9，比较9和10，符合小的在前，大的在后，顺序不变；
* 继续移动到下一个索引，扑克牌10，比较10和3，不符合从小到大的原则，进行位置交换，交换后，集合变为*[4， 9， 3， 10]*。

集合遍历第一遍后，往往很难使得集合达到预期的目标，但是对于上述集合来说，最大的扑克牌10，已经冒泡移动到了集合的最末端。

接下来进行第二遍遍历，此时比较扑克牌4和9:

![](/assets/Data-Structures-&-Algorithms-in-Swift/17/example_step.png)

只有当集合不用进行再进行交换的时候，整个集合才算所排序完成。最差的情况下，堆集合的遍历需要*n - 1*次，其中 *n* 为集合元素的个数。

## 算法实现

```swift
public func bubbleSort<Element>(_ array: inout [Element]) where Element: Comparable {
    guard array.count >= 2 else {
        return
    }
    
    for end in (1 ..< array.count).reversed() {
        var swapped = false
        
        for current in 0 ..< end {
            if array[current] > array[current + 1] {
                array.swapAt(current, current + 1)
                swapped = true
            }
        }
        if !swapped {
            return
        }
    }
}
```

1. 对集合进行元素个数检查，如果元素的个数小于2，则不需要进行排序；
2. 进行外层循环，首次循环之后，最大的元素将会移至集合的末尾，下次循环的时候，总是会比上一次少一个元素，因此，每次循环基本上都会少一次比较；
3. 进行元素间的比较和交换。比较当前元素和下一个元素的大小，如果当前元素大于下一个元素，则进行位置的交换；
4. 如果再无元素需要交换，则说明集合已经排序完成，排序退出。

```swift
example(of: "bubble sort") {
    var array = [9, 4, 10, 3]
    print("Original: \(array)")
    bubbleSort(&array)
    print("Bubble sorted: \(array)")
}

/*
---Example of bubble sort---
Original: [9, 4, 10, 3]
Bubble sorted: [3, 4, 9, 10]
*/
```

冒泡排序最好的时间复杂度为O(n)，最差时间复杂度为O(n^2)。

## 选择排序（Selection sort）

选择排序遵循冒泡排序的基本思想，但是优化了元素位置交换的数量，选择排序仅在每次传递结束之后才进行元素的交换。

### 示例

假设有如下数量的扑克牌：

![](/assets/Data-Structures-&-Algorithms-in-Swift/17/example.png)

每轮传递之后，选择排序将找到最小的未排序的元素，并对其进行位置交换：

1. 首先，发现扑克3是最小的，因此和扑克9交换位置；
2. 下一个最小的扑克是4，其已经在正确地位置；
3. 最后，最小的未扑克9，和扑克10交换。

![](/assets/Data-Structures-&-Algorithms-in-Swift/17/selection-sort-02.png)


### 算法实现

```swift
public func selectionSort<Element>(_ array: inout [Element]) where Element: Comparable {
    guard array.count >= 2 else {
        return
    }
    
    for current in 0 ..< (array.count - 1) {
        var lowest = current
        
        for other in (current + 1) ..< array.count {
            if array[lowest] > array[other] {
                lowest = other
            }
        }
        if lowest != current {
            array.swapAt(lowest, current)
        }
    }
}
```

1. 遍历除了集合最后一个元素之外的其他元素，因为如果其他的元素都在正确地位置了，那么最后一个元素也是正确位置了；
2. 再次遍历除当前索引之前的其他所有元素，寻找子集合中最小值的元素；
3. 如果当前元素的索引并不是最小元素对应的索引，则进行元素位置的交换。

```swift
example(of: "selection sort") {
    var array = [9, 4, 10, 3]
    print("Original: \(array)")
    selectionSort(&array)
    print("Selection sorted: \(array)")
}

/*
---Example of selection sort---
Original: [9, 4, 10, 3]
Selection sorted: [3, 4, 9, 10]
*/
```

类似冒泡排序，选择排序的最好、最坏和平均时间复杂度为O(n^2)，虽然有点让人沮丧，但是相比冒泡排序而言，选择排序的确表现的更好一些。

## 插入排序

插入排序是更加有用的排序算法。像冒泡排序和选择排序，插入排序的平均时间复杂度依然为O(n^2)，但是插入排序的性能不同。越多的数据需要进行排序，选择排序会带来事半功倍的效果。在集合已经排序好的情况下，插入排序能达到最好时间复杂度O(n)。在Swift标准库中的排序算法使用的是混合排序的方式，当未排序的区间元素个数小于20个元素的时候，会采用插入排序的方式。

### 示例

同上，加入有如下的扑克牌：

![](/assets/Data-Structures-&-Algorithms-in-Swift/17/example.png)

插入排序将会从左至右遍历扑克一次，每张扑克均向左移动，直到其所在位置正确位置：

![](/assets/Data-Structures-&-Algorithms-in-Swift/17/insertion-sort-02.png)

1. 在遍历时，最左边的扑克可以忽略，因为在其前面再无其他扑克牌；
2. 接下来，比较扑克9和扑克4，因为扑克4较小，因此和扑克9进行位置交换；
3. 扑克10此时不需要移动，因为再其前面的扑克为9，说明扑克10在正确地位置上；
4. 最后，扑克3前面的所有扑克均比扑克3大，因此一次交换扑克3到最首位置。

插入排序最佳的时间复杂度为O(n)，其发生在进行排序的集合元素预先是排序好的，这样在进行插入排序的时候无序进行任何左移操作。

### 算法实现

```swift
public func insertionSort<Element>(_ array: inout [Element]) where Element: Comparable {
    guard array.count >= 2 else {
        return
    }
    
    for current in 1 ..< array.count {
        for shifting in (1 ... current).reversed() {
            if array[shifting] < array[shifting - 1] {
                array.swapAt(shifting, shifting - 1)
            }else {
                break
            }
        }
    }
}
```

1. 插入排序需要遍历集合中的每一个元素，因此第一个for循环从左至右遍历集合，这里忽略了首个位置的元素；
2. 从当前索引向前遍历元素，比较当前索引元素和前一个索引元素，如果位置不正确则进行位置交换；
3. 一直交换，直到所有索引位置的元素均在正确地位置为止。如果位置正确，则跳出当前循环，进行下一个索引元素的检查。

```swift
example(of: "insertion sort") {
    var array = [9, 4, 10, 3]
    print("Original: \(array)")
    insertionSort(&array)
    print("Insertion sorted: \(array)")
}

/*
---Example of insertion sort---
Original: [9, 4, 10, 3]
Insertion sorted: [3, 4, 9, 10]
*/
```

## 算法泛化

在本内容中，将对现有的排序算法进行优化，因为现有的算法接受的集合仅仅为*Array*，对于其他的集合类型并不适用，因此将对算法进行升级，有增强算法的泛化能力，具体有如下三方面：

1. 对于插入排序而言，需要对集合进行前向遍历和元素交换，因此接受的参数集合应该是双向集合类型*BidirectionalCollection*；
2. 冒泡排序和选择排序仅仅对集合进行从前向后的遍历，因此集合参数仅仅需要符合集合类型*Collection*；
3. 无论哪一种情况下，集合必须是*MutableCollection*可变集合类型，因为需要在遍历过程中进行元素的交换；

**优化后的冒泡排序**

```swift
public func bubbleSort<T>(_ collection: inout T) where T: MutableCollection, T.Element: Comparable {
    guard collection.count >= 2 else {
        return
    }
    
    for end in collection.indices.reversed() {
        var swapped = false
        var current = collection.startIndex
        while current > end {
            let next = collection.index(after: current)
            if collection[current] > collection[next] {
                collection.swapAt(current, next)
                swapped = true
            }
            current = next
        }
        if !swapped {
            return
        }
    }
}
```

**优化后的选择排序**

```swift
public func selectionSort<T>(_ collection: inout T) where T: MutableCollection, T.Element: Comparable {
    guard collection.count >= 2 else {
        return
    }
    
    for current in collection.indices {
        var lowest = current
        var other = collection.index(after: current)
        while other < collection.endIndex {
            if collection[lowest] > collection[other] {
                lowest = other
            }
            other = collection.index(after: other)
        }
        if lowest != current {
            collection.swapAt(lowest, current)
        }
    }
}
```

**优化后的插入排序**

```swift
public func insertionSort<T>(_ collection: inout T) where T: BidirectionalCollection & MutableCollection, T.Element: Comparable {
    guard collection.count >= 2 else {
        return
    }
    
    for current in collection.indices {
        var shifting = current
        while shifting > collection.startIndex {
            let previous = collection.index(before: shifting)
            if collection[shifting] < collection[previous] {
                collection.swapAt(shifting, previous)
            } else {
                break
            }
            shifting = previous
        }
    }
}
```

## 关键点总结

* n²算法通常名声不太那么好，在性能消耗方面总是会带来更大消耗，但是在合理的数据量下，此类算法也可解决一些排序问题；
* 插入排序是最好的排序算法之一，在进行排序之前，需要了解数据是否已经是排序的。