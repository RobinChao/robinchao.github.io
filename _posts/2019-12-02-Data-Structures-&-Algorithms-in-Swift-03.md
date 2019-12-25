---
layout: post
author: Robin
title: \#3\ 关于时间复杂度和大O符号
tags: 数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/3/cover.jpg'
---

算法复杂度的衡量问题在软件开发的诞生早起就已经被提出来，并且有多个方面具体的问题。例如，从系统的架构来看，可伸缩性的架构设计和算法实现，应用程序是如何在数据特征增加的情况下被合理有效地激活的；从数据库的角度来看，数据库的处理能力是否能够应对越来越多的数据和用户行为等。

对于算法而言，可伸缩性指的是算法是否能够随着输入体量的变化，算法在执行时间和内存使用上的变现。

当你面对的是小体量的数据输入时，算法可能运行良好，执行快速，内存使用良好。但是随着数据输入体量的增加，算法的表现可能越来越糟糕，但是具体有多糟糕呢？掌握如何衡量一个算法的优劣是程序开发者的一项重要的技能。

在本篇内容中，我们将从两个角度 --- 时间维度和内存使用维度 观察算法的[大O符号](https://en.wikipedia.org/wiki/Big_O_notation)问题。

## 时间复杂度

对于小体量的数据来说，大多数的既定算法能够即快速且高效地在目标设备环境中执行，但是随着数据量的增大、业务逻辑的变化，算法的表现可能会越来越差。**时间复杂度（Time complexity）**是衡量一个算法随着输入大小的改变，其运行耗时的衡量标准，时间复杂度本质上是一个函数，一个关于输入大小和耗时之间的相关性模型。

### 恒定时间复杂度

**恒定时间**复杂度指的是，算法的执行耗时并不会随着输入体量的改变而改变。例如：

```swift
func checkFirst(names: [String]) {
    if let first = names.first {
        print(first)
    }else{
        print("no names")
    }
}
```

对于上述函数来说，输入`names`的大小并不会影响该函数的执行时间，无论`names`中有10个元素还是10万个元素，该函数仅仅检查数组中的第一个元素。对于恒定时间复杂度的算法来说，其时间复杂度可视化后如下图：

![](/assets/Data-Structures-&-Algorithms-in-Swift/3/contant-time.png)

在程序员的时间，通常使用[大O符号](https://en.wikipedia.org/wiki/Big_O_notation)来表示一个算法的时间复杂度，对于恒定时间复杂度的算法，表示为**O(1)**。

### 线性时间复杂度

假设有如下的一个函数：

```swift
func printNames(names: [String]) {
    for name in names {
        print(name)
    }
}
```

该函数打印字符串数组中的每一个元素。随着输入体量的增加，`for`循环的次数也随之增加，并且两者之间呈现线性的关系。线性时间复杂度的图像可表示为：

![](/assets/Data-Structures-&-Algorithms-in-Swift/3/linear-time.png)

线性时间复杂度相对较为简单且易于理解。当输入的数据体量增大时，算法的执行耗时会同时增加，这也是其图像是一条直线的原因。对于线性时间复杂度的算法，大O符号表示为**O(n)**。

### 二次时间复杂度

**二次时间复杂度**通常也被称为**n平方时间复杂度**，是指算法的执行耗时随着输入体量的增加而呈现二次方。例如下方示例代码：

```swift
func printNames(names: [String]) {
    for _ in names {
        for name in names {
            print(name)
        }
    }
}
```

上述示例代码的耗时是数组遍历中再次对数组进行全量遍历的时间总和。如果原始数组中有10个元素，则会打印10个元素10次，总共100次打印操作。

如果输入增加一个单位，则上述打印操作需要执行 11 * 11 次，即总共121次。可视化后的图像如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/3/quadratic-time.png)

使用大O符号表示为**O(n^2)**。

### 对数时间复杂度

到目前为止，已经了解了线性和二次时间复杂性，其中输入的每个元素至少检查一次。但是，在某些情况下，只需要检查输入的子集，从而加快运行速度。

属于此类别的时间复杂性的算法是可以通过对输入数据进行一些假设来利用一些快捷方式的算法。例如，如果您有一个已排序的整数数组，那么查找是否存在特定值的最快方法是什么？

一个普遍的解决方案是从头到尾检查数组，在得出结论之前检查每个元素，由于您检查每个元素一次，这将是一个 **O（n）** 算法，线性时间相当不错，但你可以做得更好，由于输入数组已排序，因此可以进行优化。例如以下代码：

```swift
let numbers = [1, 3, 56, 66, 68, 80, 99, 105, 450]

func naiveContains(_ value: Int, in array: [Int]) -> Bool {
    for element in array {
        if element == value {
            return true
        }
    }
    return false
}
```

如果你检查458是否在上述数组中的时候，算法将会遍历数组中的每一个元素。假设数组是已经排序好的，你可以尝试使用二分查找的方式，提高算法的执行效率，例如：

```swift
func naiveContains(_ value: Int, in array: [Int]) -> Bool {
    guard !array.isEmpty else { return false }
    let middleIndex = array.count / 2
    
    if value <= array[middleIndex] {
        for index in 0 ..< middleIndex {
            if array[index] == value {
                return true
            }
        }
    }else{
        for index in middleIndex ..< array.count {
            if array[index] == value {
                return true
            }
        }
    }
    return false
}
```

上述算法仅仅是进行了一个小的优化，即可将耗时减小一半，说明该优化是有意义的。

该算法首先检查数组的中间值，如果中间值大于目标值，曾说明目标值在数组的前半部分，否则在后半部分。每次只需要检查原有数组的一半的位置即可，这样即节省了内存空间，而且从算法的执行效率或者算法的响应能力上来说，算是一个成功的算法优化。

该算法可以重复有效地丢弃一半的数据，从而提高算法执行效率。对数时间复杂度可视化可表示为：

![](/assets/Data-Structures-&-Algorithms-in-Swift/3/logarithmic-time.png)

随着输入的增加，耗时的增加相对比较缓慢。如果仔细观察该图像，可以发现耗时呈现不温不火的现象，因为在算法的具体执行中，输入的一半已经被丢弃，算法并不关心他们。

如果你有100个元素，那么算法最终会压缩到50个元素进行检索，如果有100000个元素，最终执行时，算法只关心50000个元素而已。数据越多，丢弃的元素也就越多，最终的执行耗时和数据的输入大小之间便呈现了如上图所示的关系。对数时间复杂度使用大O符号表示为**O(log n)**。

### 准线性时间复杂度

另一个常见时间复杂度是**准线性时间复杂度**。准线性时间算法的性能比线性时间差，但明显优于二次时间复杂度。在Swift中典型的算法是数组的**sort**算法。

准线性时间复杂度的大O表示是**O（n log n）**，它是线性和对数时间的乘积。因此，准线性拟合与对数时间与线性时间不相契合；它比线性时间差一个量级，但仍比接下来您将看到的许多其他复杂性要好。下图：

![](/assets/Data-Structures-&-Algorithms-in-Swift/3/quasilinear-time.png)

准线性时间复杂性与二次时间有着类似的曲线，但对大型数据集的弹性更大。

### 其他时间复杂度

上述五种时间复杂度是程序开发中经常遇到的，还有其他的一些时间复杂度，例如多项式时间、指数时间、因子时间等。但是需要说明的是，时间复杂度并不能判断算法的执行速度，两种算法可能具有相同的时间复杂度，但是其中一种可能仍比其他算法快得多，对于小型数据集，时间复杂度可能不是实际算法执行时间的准确衡量。

例如，如果数据集较小，则插入排序等二次算法可能比准线性算法（如合并排序）更快。这是因为插入排序不需要分配额外的内存来执行算法，而合并排序需要分配多个数组。对于小型数据集，相对于算法需要接触的元素数，内存分配可能会非更加昂贵。

## 时间复杂度的比较

假设你编写了一个求从 1 到 n 和的算法，如下：

```swift
func sumFromOne(upto n: Int) -> Int {
    var result = 0
    for i in 1 ... n {
        result += i
    }
    return result
}

sumFromOne(upto: 10000)
```

上述代码中的循环将执行10000次，最终得到结果50005000。该算法是O(n)时间复杂度。但是如何改进一下该算法如下：

```swift
func sumFromOne2(upto n: Int) -> Int {
    return (1 ... n).reduce(0, +)
}

sumFromOne2(upto: 10000)
```

解决同样的问题，但是该实现的执行上会比上面循环的代码快很多，但是这里的时间复杂度依然是O(n)。使用`reduce`时，系统内部会执行 `n`次加法，但是调用的是Swift标准库中已经编译的代码，因此省去了很大一部分代码的编译时间。

继续优化上述代码，如下：

```swift
func sumFromOne3(upto n: Int) -> Int {
    return (n + 1) * n / 2
}

sumFromOne3(upto: 10000)
```

整个版本的算法使用了弗雷德里克·高斯算法，可以使用简单的算术计算总和。该算法的最时间复杂度是**O（1）**，属于恒定时间算法。也是该特定问题在时间复杂度上的的最优算法。


## 空间复杂度

算法的时间复杂度有助于预测算法的可伸缩性，但它并不是唯一的指标。**空间复杂性是算法运行所需的资源的度量。** 对于计算机而言，内存一直是昂贵而紧俏的资源。假设有以下代码：

```swift
func printSorted(_ array: [Int]) {
    let sorted = array.sorted()
    for element in sorted {
        print(element)
    }
}
```

上述代码创建了一个排序后的数组拷贝并遍历该数据，打印其中元素。为了计算其空闲复杂度，需要分析该函数的内存开辟情况。

**array.sorted()**方法的调用系统会新建一个和原数组同样大小和类型的新数组，因此`printSorted`函数的空间复杂度是 **O(n)**。当然对于在内存中开辟尽量小的空间而言，该函数是简单而轻量的。可以将上述函数修改为如下方式：

```swift
func printSorted2(_ array: [Int]) {
    // 1
    guard !array.isEmpty else { return }
    
    // 2
    var currentCount = 0
    var minValue = Int.min
    
    // 3
    for value in array {
        if value == minValue {
            print(value)
            currentCount += 1
        }
    }
    
    while currentCount < array.count {
        // 4
        var currentValue = array.max()!
        
        for value in array {
            if value < currentValue && value > minValue {
                currentValue = value
            }
        }
        
        // 5
        for value in array {
            if value == currentValue {
                print(value)
                currentCount += 1
            }
        }
        
        // 6
        minValue = currentValue
    }
}
```

此实现尊重空间限制。总体目标是多次遍历迭代数组，为每个迭代打印下一个最小值。

1. 检查是否数组为空的情况。如果是，则不打印内容。

2. `currentCount`跟踪打印语句的数量。`minValue` 存储最后一个打印值。

3. 算法首先打印出与 `minValue`  匹配的所有值，并根据打印语句的数量更新当前计数。

4. 使用 `while` 循环，算法查找大于 `minValue`  的最小值并将其存储在当前值中。

5. 然后，该算法在更新`currentCount`的同时打印数组内所有`currentValue`的值。

6. `minValue` 设置为`currentValue`，因此下一次迭代将尝试查找下一个最小值。

上述算法仅分配内存以跟踪几个变量，因此空间复杂性为 **O(1)**。这与前面的函数不同，后者分配整个数组以创建源数组的排序表示形式。

> PS: 在实际开发中并不会为了追求类似的空间，而将代码写成上述样子，上述代码仅仅是为了说明代码的不同写法，会导致算法的空间复杂度有质的飞跃。

## 关键点总结

* **时间复杂度**是对输入大小增加时，算法运行时间的衡量；
* **空间复杂度**是对算法运行时，对资源使用情况的衡量；
* **大O符号表示法**是用于表示时间和空间复杂性的一般形式；
* 时间和空间复杂性是可伸缩性的高级度量，它们不测量算法本身的实际速度；
* 对于小型数据集，时间复杂性通常无关紧要。准线性算法可能比线性算法慢。