---
layout: post
author: Robin
title: \#18\ 归并排序（Merge Sort）
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/18/cover.jpg'
---

**归并排序[Merge Sort]**是最有效的排序算法之一，它的时间复杂度为O(n log n)，是所有通用排序算法中速度最快的一种。归并排序背后的思想是**分而治之**，即将一个大问题分解成多个更小、更易于解决的问题，然后将各个小问题的结果合并为最终结果。**归并排序的终极秘诀是先拆分后合并。**

例如，有如下未排序的扑克牌：

![](/assets/Data-Structures-&-Algorithms-in-Swift/18/eg-1.png)

针对上述问题，归并排序的工作原理如下：

* 对扑克牌进行对半拆分，拆分后会有两大部分：

![](/assets/Data-Structures-&-Algorithms-in-Swift/18/eg-2.png)

* 对上一步的拆分结果继续进行拆分：

![](/assets/Data-Structures-&-Algorithms-in-Swift/18/eg-3.png)

直到无法再拆分为止

![](/assets/Data-Structures-&-Algorithms-in-Swift/18/eg-4.png)

* 最后，将拆分的每一部分进行反向的合并，每次合并时，将不同的部分按照顺序进行排序。

![](/assets/Data-Structures-&-Algorithms-in-Swift/18/eg-5.png)

## 算法实现

归并排序的思想是分而治之，因此首先要进行问题的拆解，之后再进行合并。

### 拆分

```swift
public func mergeSort<Element>(_ array: [Element]) -> [Element] where Element: Comparable {
    let middle = array.count / 2
    let left = Array(array[..<middle])
    let right = Array(array[middle...])
    
    // ... more to come
}
```

首先对待排序集合进行了对半拆分，但是仅仅一次拆分并不能满足归并排序的思想，需要持续拆分，直到集合再无法进行拆分为止，因此更新上述代码如下：

```swift
public func mergeSort<Element>(_ array: [Element]) -> [Element] where Element: Comparable {
    guard array.count > 1 else {
        return array
    }
    
    let middle = array.count / 2
    let left = mergeSort(Array(array[..<middle]))
    let right = mergeSort(Array(array[middle...]))
    
    // ... more to come
}
```

* 首先确定算法可进入的条件，如果不符合基本的条件，则算法退出。这里的退出条件是待排序集合中仅有一个元素或无元素时，算法退出；
* 使用递归的方式进行拆分，直至无法再次拆分为止。

### 合并

完全拆分后，则进入到了归并排序的最后一步，将所拆分的左右部分进行合并，此时新建函数*merge*进行合并操作：

```swift
private func merge<Element>(_ left: [Element], _ right: [Element]) -> [Element] where Element: Comparable {
    var leftIndex = 0
    var rightIndex = 0
    var result: [Element] = []
    
    while leftIndex < left.count && rightIndex < right.count {
        let leftElement = left[leftIndex]
        let rightElement = right[rightIndex]
        
        if leftElement < rightElement {
            result.append(leftElement)
            leftIndex += 1
        } else if leftElement > rightElement {
            result.append(rightElement)
            rightIndex += 1
        } else {
            result.append(leftElement)
            leftIndex += 1
            result.append(rightElement)
            rightIndex += 1
        }
    }
    if leftIndex < left.count {
        result.append(contentsOf: left[leftIndex...])
    }
    if rightIndex < right.count {
        result.append(contentsOf: right[rightIndex...])
    }
    return result
}
```

1. *leftIndex*和*rightIndex*两个变量用于对遍历过程进行索引追踪；
2. *result*变量为最终的合并结果；
3. 使用*while*循环对左右集合进行遍历和元素比较，直到到达集合末尾位置；
4. 在遍历过程中，对元素进行比较，更小的元素或相等的元素均将追加到结果*result*并对索引进行移动；
5. *while*循环结束后，*left*和*right*两个集合均是已经排序了的，确保了剩下的元素都是大于或等于*result*中已存在的元素。此种情况下，可以直接将其追加到结果集合中。

### 整合

合并工作使用了单独的函数完成，此时整合拆分和合并，最终归并排序算法如下：

```swift
public func mergeSort<Element>(_ array: [Element]) -> [Element] where Element: Comparable {
    guard array.count > 1 else {
        return array
    }
    
    let middle = array.count / 2
    let left = mergeSort(Array(array[..<middle]))
    let right = mergeSort(Array(array[middle...]))
    return merge(left, right)
}
```

总结一下归并排序的关键流程：

1. 归并排序的核心思想是分而治之，即将大问题拆分成多个小问题，依次对各个小问题进行求解，最后在合并各个结果；
2. 归并排序算法有两个核心的职责：一个是递归拆分初始集合的方法，另一个是合并两个集合的方法；
3. 合并函数应该使用两个排序的数组并生成一个排序的数组。

```swift
example(of: "merge sort") {
    let array = [7, 2, 6, 3, 9]
    print("Original: \(array)")
    print("Merge sorted: \(mergeSort(array))")
}

/*
---Example of merge sort---
Original: [7, 2, 6, 3, 9]
Merge sorted: [2, 3, 6, 7, 9]
*/
```

## 性能

归并排序有着并不太坏的时间复杂度，其最好、最坏和平均时间复杂度为O(n log n)。

* 在递归的时候，需要将一个集合拆分成更小的集合，这意味着大小为2的集合需要一次递归，大小为4的集合需要两次递归，大小为8的集合需要三次递归等等。一般情况下，如果集合的大小为n，则要拆分的层级数就是log2(n)；
* 一个递归级别将合并n个元素。不管合并的规模是大是小;每一层合并的元素数量仍然是n。这意味着一个递归的代价是O(n)。

那么拆分和合并的总体消耗为 *O(log n) x O(n) = O(n log n)*。

## 关键点总结

* 归并排序的核心思想是**分而治之**的原则；
* 归并排序算法的实现由多种方式，不同的实现可能带来不同的性能体现。
