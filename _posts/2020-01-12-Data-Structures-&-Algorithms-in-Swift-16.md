---
layout: post
author: Robin
title: \#16\ 优先级队列
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/16/cover.png'
---

队列是一种先进先出（FIFO）的数据结构，而优先级队列是另一种队列结构，其可替代队列的先进先出顺序，该队列中的元素有着优先级的顺序。一个优先级队列也可以是：

1. **最大优先级队列：**队列中最前面的元素具有最高优先级；
2. **最小优先级队列：**队列中最前面的元素具有最低优先级。

当需要在给定的元素列表中取标定最大元素和最小元素时，优先级队列将是非常合适的一种数据结构。

## 优先级队列的典型应用

* **迪克斯特拉的算法（Dijkstra’s algorithm）**，使用优先级队列计算最小代价。
* **A\*路径寻找算法**，使用优先级队列跟踪对位置路径进行探索的最短路径。
* **堆排序**，可以使用优先级队列实现。
* **哈夫曼编码**会构建一个压缩树。最小优先级队列用于重复查找两个频率最小的节点，这些节点未具有父节点。

优先级队列的应用范围很广，远不止上述列举的部分。

## 一般操作

在 [\#8\\ 队列的Swift实现与操作定义](https://robinchao.github.io/Data-Structures-&-Algorithms-in-Swift-08/)中我们为队列定义了如下的一个协议：

```swift
public protocol Queue {
    associatedtype  Element
    mutating func enqueue(_ element: Element) -> Bool
    mutating func dequeue() -> Element?
    var isEmpty: Bool { get }
    var peek: Element? { get }
}
```

优先级队列和普通队列同样有着相同的操作，只是具体的实现会有所不同。

对于优先级队列而言，同样对遵循协议**Queue**，并实现一些一般性的操作如下：

* **enqueue：**插入一个元素到队列，如果操作成功，则返回*true*；
* **dequeue：**移除具有最高优先级的元素，并返回它，如果队列为空，则返回*nil*；
* **isEmpty：**检查队列是否为空；
* **peek：**返回具有最高优先级的队列，但并不进行删除，如果队列为空，则返回*nil*。

## 算法实现

构建优先级队列的方式有以下几种：

1. **已排序数组**：在获取最大值或最小值的时间复杂度均为O(1)，使用此数据构建优先级队列非常有效，但是其插入算法却比较慢，会达到O(n)的时间复杂度。
2. **平衡二叉搜索树**：在创建双端优先级队列时，使用平衡二叉搜索树最为有利，此时获取最小值和最大值的时间复杂度均在_O(log n)，插入算法比排序的数组会更好，在O(log n)。
3. **堆**：优先级队列最佳的选择，堆结构比排序的数组更为有效，因为堆只需要部分排序，除了从最小堆中获取最小值和从最大堆中获取最大值为O(1)的快速外，其他的操作均为O(log n)的时间复杂度。

```swift
struct PriorityQueue<Element: Equatable>: Queue {
    private var heap: Heap<Element>
    
    init(sort: @escaping (Element, Element) -> Bool, elements: [Element] = []) {
        heap = Heap(sort: sort, elements: elements)
    }
}
```

1. *PriorityQueue* 遵循队列协议*Queue*。泛型参数元素必须遵循*Equatable*，因为在元素的操作中需要能够进行元素间的比较。
2. 使用堆数据结构实现优先级队列；
3. 传递合适的参数到初始化构造函数，*PriorityQueue*可根据参数构建最小和最大优先级队列。

为了遵循*Queue*协议，需要增加如下的协议方法：

```swift
public var isEmpty: Bool {
    return heap.isEmpty
}

public var peek: Element? {
    return heap.peek()
}

public mutating func enqueue(_ element: Element) -> Bool {
    heap.insert(element)
    return true
}

public mutating func dequeue() -> Element? {
    return heap.remove()
}
```

在实践中优先级队列上，堆是最为完美的选择，只需要调用堆的各种方法即可实现优先级队列的各种操作。

## 测试

```swift
example(of: "priorityQueue") {
    var priorityQueue = PriorityQueue(sort: >, elements: [1, 12, 3, 4, 1, 6, 8, 7])
    while !priorityQueue.isEmpty {
        print(priorityQueue.dequeue()!)
    }
}

/*
---Example of priorityQueue---
12
8
7
6
4
3
1
1
*/
```

## 关键点总结

* 一个优先级队列通常使用**优先级顺序**进行元素的查找；
* 能够通过关注队列的关键操作而排除堆数据结构提供的其他功能，从而创建抽象层。
* 使得优先级队列的意图清晰而简洁。唯一的工作是排队和取消排队元素。
