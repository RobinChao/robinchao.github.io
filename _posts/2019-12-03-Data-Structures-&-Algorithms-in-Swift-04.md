---
layout: post
author: Robin
title: \#4\ 单向链表的Swift实现
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/4/cover.jpg'
---

**链表**是一种线性的、单向的数据结构，不同于数组连续的内存存储，链表中的元素在内存是独立的对象。链表具有以下理论优势：

* 元素插入和从列表头部删除元素的时间恒定；
* 具有可靠的性能特性。

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/a-linked-list.png)

如上图所示，链表的结构是一个**节点**结构。节点具有两个功能：

1. 保存值；
2. 保存下一个节点的指针。**nil**节点表示链表的结尾。

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/linked-list-node.png)

> 链表分为**单向链表**和**双向链表**，双向链表的每个阶段，还具有指向前一个节点的指针。

在本文中，将学习链表的Swift实现，以及关于链表的一些通用的操作。将了解每个操作的时间复杂性，并实现一个整洁的小 Swift 功能，称为**写入时复制**。

## 节点（Node）

首先我们需要定义**节点**的数据结构，使用Xcode的新建Playground，并在**Sources**目录下新建**Node.swift**文件。根据对节点功能的了解，在Node的结构中，应该有至少两个数据属性，一个用来存储，一个用于指向，如下：

```swift
public class Node<Value> {
    public var value: Value
    public var next: Node?
    
    public init(value: Value, next: Node? = nil) {
        self.value = value
        self.next = next
    }
}

extension Node: CustomStringConvertible {
    public var description: String {
        guard let next = next else {
            return "\(value)"
        }
        return "\(value) ->" + String(describing: next) + " "
    }
}
```

以上便定义好了一个单向链表的节点结构。为了能够在测试方便，可以添加如下工具函数：

> 新建helper.swift文件，编写工具类函数：
> 
> ```swift
> public func example(of description: String, action: () -> Void) {
>  print("---Example of \(description)---")
>  action()
>  print()
>}
> ```

此时，回到主Playground，定义一个链表结构的示例，如下：

```swift
example(of: "creating and linking nodes") {
    let node1 = Node(value: 1)
    let node2 = Node(value: 2)
    let node3 = Node(value: 3)
    
    node1.next = node2
    node2.next = node3
    
    print(node1)
}

// ---Example of creating and linking nodes---
// 1 ->2 ->3 
```

示例所定义的链表结构，图示如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/demo-linked-list-1.png)

一个单向链表，包含了三个值分别为1、2、3的节点。这样的链表定义似乎已经完成了，但是当节点的数量上升后，我们就需要每个节点都进行赋值和指针指向的定义，显然这样的方式即笨重且不灵活。就实用性而言，目前的结构还需要进行优化，缓解此问题的常用方法是构建一个专门管理Node节点对象的链表结构，如下：

## LinkedList

**Sources**目录下新建**LinkedList.swift**文件。

```swift
public struct LinkedList<Value> {
    public var head: Node<Value>?
    public var tail: Node<Value>?
    
    public init() {}
    
    public var isEmpty: Bool {
        return head == nil
    }
}


extension LinkedList: CustomStringConvertible {
    public var description: String {
        guard let head = head else {
            return "Empty Linked List"
        }
        return String(describing: head)
    }
}
```

该结构下的链表具有**头**和**尾**概念，分别指向链表的第一个节点和末尾节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/head-tail.png)

## 向链表中添加值

有了节点管理形式的链表后，需要为该链表**添加**头结点，**追加**末尾节点，以及**插入**节点，因此需要定义三个通用的方法：

* **push**：在链表的头部添加值；
* **append**：在链表的末尾添加值；
* **insert(after:)**：在链表的特定节点后添加值。

### push操作

在链表的头部位置添加值，也就是**头插法**，其时间复杂度为**O(1)**。其实现较为简单，如下：

```swift
public mutating func push(_ value: Value) {
    head = Node(value: value, next:head)
    if tail == nil {
        tail = head
    }
}
```

> **struct** 和 **enum** 的值在内部是不能修改的, 如果要修改需要在方法前面添加 **mutating** 修饰符

对于一个空的链表来说，头节点也是尾节点，采用头插法插入值后，每次插入的值都会放在链表的头部。

回到主Playground，进行链表的**push**操作：

```swift
example(of: "push") {
    var list = LinkedList<Int>()
    
    list.push(3)
    list.push(2)
    list.push(1)
    
    print(list)
}

// ---Example of push---
// 1 ->2 ->3 
```

### append操作

在链表的尾部位置添加值，也就是**尾插法**，其时间复杂度**O(1)**。实现如下：

```swift
public mutating func append(_ value: Value) {
    // 如果链表尾空，使用push操作新建节点，更新链表的头节点和尾节点。
    guard !isEmpty else {
        push(value)
        return
    }
    // 创建一个新节点，赋值为尾部节点的下一个节点，将节点连接起来。
    tail!.next = Node(value: value)
    // 因为是尾部拼接节点，所以新的节点将成为尾部节点
    tail = tail!.next
}
```

回到主Playground，进行链表的**append**操作：

```swift
example(of: "append") {
    var list = LinkedList<Int>()
    
    list.append(1)
    list.append(2)
    list.append(3)
    
    print(list)
}

// ---Example of append---
// 1 ->2 ->3 
```

### insert(after:) 操作

**插入**操作是指在链表的特定节点位置，插入一个新的节点。该操作需要两个步骤完成：

1. 查找特定节点
2. 插入新的节点

首先需要根据给定的索引，查找特定的节点：

```swift
public func node(at index: Int) -> Node<Value>? {
    // 由于只能从头部遍历链表，因此先创建当前节点和索引的引用
    var currentNode = head
    var currentIndex = 0
    // 使用while循环，将引用向下移动到列表中，直到达到所需的索引。 空列表或越界索引将导致nil返回值。
    while currentNode != nil && currentIndex < index {
        currentNode = currentNode!.next
        currentIndex += 1
    }
        
    return currentNode
}
```

这里需要说明的是，我们针对的是**单向链表**，链表节点的访问只能从头节点依次向后访问，在实现节点的查找时，必须使用迭代的方式进行节点遍历。

找到了特定索引的节点后，接下来就是插入一个新的节点了。

```swift
@discardableResult
public mutating func insert(_ value: Value, after node: Node<Value>) -> Node<Value> {
    // 如果要插入新节点的位置是尾节点，则直接使用append操作，添加新的尾节点
    guard tail !== node else {
        append(value)
        return tail!
    }
        
    // 否则新建节点，并赋值为查找节点的下一个节点，将节点进行连接
    node.next = Node(value: value, next: node.next)
        
    return node.next!
}
```

> **@discardableResult**指的是接口的调用者忽略此方法的返回值，而编译器不会向上和向下跳过警告。

完成逻辑编写后，回到主Playground，进行链表的**insert(after:)**操作：

```swift
example(of: "inserting at a particular index") {
    var list = LinkedList<Int>()
    
    list.push(3)
    list.push(2)
    list.push(1)
    
    print("Before inserting: \(list)")
    
    let middleNode = list.node(at: 1)!
    for _ in 1 ... 3 {
        list.insert(-1, after: middleNode)
    }
    print("After inserting: \(list)")
}

// ---Example of inserting at a particular index---
// Before inserting: 1 ->2 ->3  
// After inserting: 1 ->2 ->-1 ->-1 ->-1 ->3 
```

### 性能分析 --- 时间复杂度

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/performance-analysis.png)

## 从链表中删除值

对应向链表中添加值，从链表中删除值同样对应三个主要的方法：

1. **pop**：从链表头部删除值；
2. **removeLast**：从链表尾部删除值；
3. **remove(after:)**：删除指定位置的节点值。

### pop 操作

从链表的头部直接删除值的操作和给链表头部添加值的操作**push**有点类似，其算法也相对简单，如下：

```swift
@discardableResult
public mutating func pop() -> Value {
    defer {
        head = head?.next
        if isEmpty {
            tail = nil
        }
    }
    return head?.value
}
```

**pop**方法的返回值是被删除的节点的值。该结果是一个**可选值**，因为链表有可能是一个空链表。

将原链表的头节点向前移动一个位置即可，在**ARC**模式下，系统内存会在头部节点无任何引用的时候，自动清理原链表的头节点资源。如果移动头节点后，链表是一个空链表，需要将尾部节点重新置空。

回到主Playground，尝试进行链表的**pop**操作：

```swift
example(of: "pop") {
    var list = LinkedList<Int>()
    
    list.push(3)
    list.push(2)
    list.push(1)
    
    print("Before popping list: \(list)")
    let poppedValue = list.pop()
    print("After popping list: \(list)")
    print("Popped value: " + String(describing: poppedValue))
}

// ---Example of pop---
// Before popping list: 1 ->2 ->3  
// After popping list: 2 ->3 
// Popped value: Optional(1)
```

### removeLast 操作

删除链表的尾部节点相对比较复杂，算法如下：

```swift
@discardableResult
public mutating func removeLast() -> Value? {
    // 如果链表的头节点为nil，无节点移除，故返回nil
    // 这里也可以直接使用isEmpty进行判断
    guard let head = head else {
        return nil
    }
    // 如果链表仅有一个节点，头节点也是末尾节点，因此直接使用pop()操作即可
    guard head.next != nil else {
        return pop()
    }
    // 创建节点的引用
    var prev = head
    var current = head
    // 遍历链表节点，直到当前节点的下一个节点为nil，则表明当前节点已经是末尾节点了
    while let next = current.next {
        prev = current
        current = next
    }
    // 将当前节点的前一个节点的next指针值为nil，并更新末尾节点
    prev.next = nil
    tail = prev
    // 返回被删除的值
    return current.value
}
```


回到主Playground，尝试进行链表的**removeLast**操作：

```swift
example(of: "removing the last node") {
    var list = LinkedList<Int>()
    
    list.push(3)
    list.push(2)
    list.push(1)
    
    
    print("Before remove last node: \(list)")
    let removedValue = list.removeLast()
    print("After remove last node: \(list)")
    print("Removed value: " + String(describing: removedValue))
}

// ---Example of removing the last node---
// Before remove last node: 1 ->2 ->3  
// After remove last node: 1 ->2 
// Removed value: Optional(3)
```

**removeLast**操作需要遍历整个链表，因此其时间复杂度为**O(n)**。

### remove(after:) 操作

**remove(after:)**操作类似**insert(after:)**，首先需要找到特定位置的节点，然后执行删除操作。算法过程如下图：

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/remove-at.png)

```swift
@discardableResult
public mutating func remove(after node: Node<Value>) -> Value? {
    defer {
        if node.next === tail {
            tail = node
        }
        node.next = node.next?.next
    }
    return node.next?.value
}
```

节点的引用清理放在**defer**区块内，当删除的节点无任何引用的时候，系统内存会自动清理其占用资源。


回到主Playground，尝试进行链表的**remove(after:)**操作：

```swift
example(of: "removing a node after a particular node") {
    var list = LinkedList<Int>()
    
    list.push(3)
    list.push(2)
    list.push(1)
    
    print("Before removing at particular index: \(list)")
    let index = 1
    let node = list.node(at: index - 1)!
    let removedValue = list.remove(after: node)
    print("After removing at index \(index): \(list)")
    print("Removed value: " + String(describing: removedValue))
}

// ---Example of removing a node after a particular node---
// Before removing at particular index: 1 ->2 ->3  
// After removing at index 1: 1 ->3 
// Removed value: Optional(2)
```

### 性能分析 --- 时间复杂度

![](/assets/Data-Structures-&-Algorithms-in-Swift/4/remove-performance-analysis.png)


## 阶段性总结

支持关于LinkedList的定义和基本操作就完成了，对于链表而言，单向链表只是一个开始，这里没有涉及到双向链表。

* 链表是有一个个节点链接而成的，每个节点具有保存值和下一个节点索引的属性；
* 链表中节点值的添加涉及头尾和指定位置，在指定位置插入需要两步操作，首先找到给定位置的节点，然后删除其后的一个节点并重建链接；
* 链表中节点值的删除同样涉及头尾和指定位置；
* 在进行链表的操作时，需要时刻记住在必须的时机更新链表的节点指向指针，保证链表的链接性。