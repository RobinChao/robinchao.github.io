---
layout: post
author: Robin
title: \#19\ 基数排序（Radix Sort）
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/19/cover.jpg'
---

**基数排序[Radix Sort]**是一种在线性时间内对整数进行排序的非比较算法。

为了简单起见，在本文中将关注以10为基数的整数排序，以及基数排序中的*最小有效位[LSD]*的变体等。

## 示例

为了进行基数排序的工作方式，假设需要对如下的集合进行排序：

```swift
var array = [88, 410, 1772, 20]
```

基数排序依赖于整数的位置表示法，如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/19/integer-base.png)

首先，按照最小有效位---个位对集合中的元素进行拆分：

![](/assets/Data-Structures-&-Algorithms-in-Swift/19/eg-1.png)

然后按照个位数从小至大的顺序对上图元素进行排序，结果如下：

```swift
array = [410, 20, 1772, 88]
```

接下来，重复上述步骤，按照十位对集合中的元素进行拆分：

![](/assets/Data-Structures-&-Algorithms-in-Swift/19/eg-2.png)

此时按照十位拆分后再进行排序后，和按照个位排序的结果相同，因此此时不进行重排。

继续按照百位堆集合中的元素进行拆解，拆解后如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/19/eg-3.png)

有一些元素可能没有百位数，或者其他位也可能没有数，此时拆解时将其赋值为0即可。按照百位重新对集合元素进行排序，结果如下：

```swift
array = [20, 88, 410, 1772]
```

最后，在堆集合中的元素进行千位拆解：

![](/assets/Data-Structures-&-Algorithms-in-Swift/19/eg-4.png)

重新按照千位拆解结果进行排序，结果如下：

```swift
array = [20, 88, 410, 1772]
```

当多个数组出现在拆解后的结果中时，则其排序不需要更改。例如在百位拆解中，20在88之前，因为在十位拆解时，20的拆解结果2和88的拆解结果8已经决定了20在88之前。

## 算法实现

```swift
extension Array where Element == Int {
    public mutating func radixSort() {
        let base = 10
        var done = false
        var digits = 1
        while !done {
            // more to come
        }
    }
}
```

基数排序针对的是整数集合，因此在算法实现中直接对集合类型Array进行扩展，并制定元素类型为Int。上述函数定义和相关变量和逻辑相对简单，具体如下：

1. 使用10为基数堆整数进行拆解和排序。因为在算法执行过程中需要多次使用这个基数，因此使用变量*base*进行存储；
2. 使用两个变量是否结束done和数字digit变量对执行过程进行追踪。基数排序在执行过程中有多次的遍历，done变量以标识整个遍历过程是否结束，digit变量用来标识当前所处理的数字。

接下来需要编写的是针对每一步进行排序的逻辑算法，可称之为**桶排序[Bucket Sort]**。

### Bucket Sort

此排序算法主要是在*while*循环体中执行，具体如下：

```swift
var buckets: [[Int]] = .init(repeating: [], count: base)
            
forEach {
    number in
    let remainingPart = number / digits
    let digit = remainingPart % base
    buckets[digit].append(number)
}

digits *= base
self = buckets.flatMap { $0 }
```

1. 使用二维数组的方式初始化buckets。因为使用的基数是10，因此拆解后会有10个buckets；
2. 对集合中的每一个元素进行拆分，并放置在对应的bucket中；
3. 使用digit的内容更新为希望检查和更新数组的的下一个数字。*flatMap*方法将二维数组变成一维数组，即将每一部分bucket排序装进数组。

**循环何时结束？**

上述实现虽然逻辑上能够很好的拆解元素，并进行排序，但是对于*while*循环并没有机会符合退出条件，因此会进入无限循环状态。要符合退出条件，添加如下条件：

1. 在*while*循环的开始，添加*done = true*；
2. 在forEach闭包结构中，增加如下语句：

```swift
if remainingPart > 0 {
    done = false
}
```

只要还有未排序的数字，*forEach*就会一直迭代，直到再无未排序的部分，*forEach*执行完毕。

```swift
example(of: "radix sort") {
    var array = [88, 410, 1772, 20]
    print("Original: \(array)")
    array.radixSort()
    print("Radix sorted: \(array)")
}

/*
---Example of radix sort---
Original: [88, 410, 1772, 20]
Radix sorted: [20, 88, 410, 1772]
*/
```

**基数排序**是最快速的排序算法之一，其平均时间复杂度为O(k*n)，其中*k*为最大数字的有效位数，*n*为数组中整数的个数。

基数排序在*k*为常数时最有效，当数组中所有数字的有效位数都相同时，基数排序最有效。它的时间复杂度变成了O(n)，基数排序也会带来O(n)空间复杂度。

## 关键点总结

* 不像之前的排序算法，基数排序是一种非比较性排序，它不依赖于两个值之间的比较。基数排序利用桶排序，桶排序类似于筛选值的筛子；

* 基数排序是最快速的排序算法之一，利用了数字的位置等；

* 本文讨论了最小有效数字基数排序。另一种实现基数排序的方法是最有效的数字形式。这种形式通过优先排列最有效的数字而不是最不重要的数字进行排序。