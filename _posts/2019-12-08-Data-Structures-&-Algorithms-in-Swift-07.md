---
layout: post
author: Robin
title: \#7\ Stack & Stack Simple Challenges
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/7/cover.jpeg'
---

**栈（Stack）**是一个常见的基础类型数据结构，在生活中经常也能看到栈的身影，例如一本书、一叠现金等等。栈的数据结构在概念上和对象的物理存储栈相同。再向栈添加元素时，需要将其放在栈顶，通俗称压栈，而从站内删除元素时，始终删除的是栈顶的元素，又称为出栈，而这种进栈和出栈的特性使得栈属于一种受限的线性表结构。栈的操作特性通常被称为**后进先出(LIFO-last in first out)**的方式，另一种数据结构队列的操作特性与栈有着不同，通常称为**先进先出(FIFO-first in first out)**。


## 栈的一般操作

栈是一种很有用，但是相对简单的数据结构。构建栈类型数据结构的主要目标是数据的访问权限和方式问题。相比于链表而言，栈并没有链表那个复杂和琐碎。

对于栈来说，主要的操作有两个，即上述所说的**压栈**和**出栈**的操作：

* **push：**添加一个元素到栈顶；
* **pop：**从栈顶删除一个元素

也就是说，对于栈而言，只能从栈的一边添加或者移除元素，也就是上述所说的**后进先出(LIFO-last in first out)**的方式。在计算机编程中，栈的身影无处不在，例如下面几个场景中，都是栈的理念和其应用的结果：

* 在iOS中,导航控制器的作用是将视图控制器的视图弹出或者弹入，并且最新弹出的总是最后弹入的视图控制器视图；
* 内存分配在体系结构级别使用堆栈。局部变量的内存也使用堆栈进行管理；
* Search和conquer算法，例如从迷宫中寻找路径，均使用堆栈来方便回溯。

## 栈数据结构实现

首先定义栈的基础结构，对于栈而言，其核心就是一个列表，只是再具体的操作时有LIFO的限制。

```swift
public struct Stack<Element> {
    private var storage: [Element] = []
    
    public init() {}
}

extension Stack: CustomStringConvertible {
    public var description: String {
        let topDivider = "---- top ----\n"
        let bottomDivider = "\n -----------"
        
        let stackElements = storage
            .map { "\($0)" }
            .reversed()
            .joined(separator: "\n")
        return topDivider + stackElements + bottomDivider
    }
}
```

另外，在栈的数据结构中使用列表的方式进行数据存储，是因为对于列表来说，在其一端进行操作 --- **append** 和**popLast**，都属于恒定时间的复杂度O(1)。也更是促进了栈的进栈和出栈特性的性能表现。

## push & pop 操作

在栈的数据结构定义中，增加基本的压栈和出栈操作，压栈操作直接使用列表的**append**，出栈使用**popLast**：

```swift
// push
public mutating func push(_ element: Element) {
    storage.append(element)
}
    
// pop
@discardableResult
public mutating func pop() -> Element? {
    return storage.popLast()
}
```

对于这两个基本操作来说，实现也非常直截了当。接下来对其进行实际测试，在主Playground中，进行测试代码编写。

> 在进行测试前，可以将[\\#4\ Linked List 的Swift实现
](https://robinchao.github.io/2019/12/03/Data-Structures-&-Algorithms-in-Swift-04.html)中的**Helper.swift**拷贝到当前工程中。

```swift
example(of: "using a stack") {
    var stack = Stack<Int>()
    stack.push(1)
    stack.push(2)
    stack.push(3)
    stack.push(4)
    
    print(stack)
    
    if let poppedElement = stack.pop() {
        assert(4 == poppedElement)
        print("Popped: \(poppedElement)")
    }
}

/* 
---Example of using a stack---
---- top ----
4
3
2
1
 -----------
Popped: 4
*/
```

## 扩展性操作

对于栈来说，除了常用的push和pop操作之外，还有一些额外的操作，能够提高对栈的使用等。

```swift
// peek
public func peek() -> Element? {
    return storage.last
}
    
// isEmpty
public var isEmpty: Bool {
    return peek() == nil
}
```

* **peek()：**获取栈顶元素
* **isEmpty：**判断栈是否为空

## Less is more

在链表的实现中，我们使用了Swift标准库中的Collection协议，那么在栈的实现中是否也能够使用Collection协议呢？栈的目的是有限制的访问数据，通过迭代或者下标的方式即可实现该目标，但是对于Collection协议来说，并不止于此，因此在栈上使用Collection协议和栈的最初目标是相互制约的。在这种情况下，少即是多！

您可能希望采用现有数组并将其转换为栈，以便保证访问顺序，也可以循环遍历数组元素以及添加元素。对于栈来说，为了能够对栈的操作有一个统一的初始化存储方式，可以定义其初始化方式：

```swift
public init(_ elements: [Element]) {
    storage = elements
}
```

```swift
example(of: "initializing a stack from a array") {
    let array = ["A", "B", "C", "D"]
    var stack = Stack(array)
    print(stack)
    
    if let poppedElement = stack.pop() {
        print("Popped: \(poppedElement)")
    }
}

/*
---Example of initializing a stack from a array---
---- top ----
D
C
B
A
 -----------
Popped: D
*/
```

上述实现中，将一个数组转化为了栈，并且栈中元素的数据类型是String，也就意味着栈中可以放置多种类型的数据元素。

既然可以使用数组直接转化为栈，那么是否可以直接使用数组的方式初始化栈呢？

```swift
extension Stack: ExpressibleByArrayLiteral {
    public init(arrayLiteral elements: Element...) {
        storage = elements
    }
}
```

```swift
example(of: "initializing a stack from an array literal") {
    var stack: Stack = [1.0, 2.0, 3.0, 4.0]
    print(stack)
    if let poppedElement = stack.pop() {
        print("Popped: \(poppedElement)")
    }
}

/*
---Example of initializing a stack from an array literal---
---- top ----
4.0
3.0
2.0
1.0
 -----------
Popped: 4.0
*/
```

在搜索树和图的问题求解中，栈至关重要。例如在查找迷宫的路径方法中，每次叨叨左、右、前或后的决策点时，都可以将所有可能的决策点压入栈中，当栈顶的路径是一个死胡同时，只需要从栈中弹出并继续下一个判断，直到走出迷宫即可。

## 关键点总结

* 栈的数据结构虽然非常简单，但栈是解决很多问题的关键性数据结构；
* 对于栈爱说，只有两个基本操作，分别是压栈**push**和出栈**pop**。

## 栈的挑战

## Challenge 1：在不适用递归的情况下，反向打印一个单向链表的节点。

> 在[\\#6\ Linked List 挑战](https://robinchao.github.io/2019/12/05/Data-Structures-&-Algorithms-in-Swift-06.html)中我们使用了递归的方式，反向打印了一个链表的节点。在这里将使用栈的结构进行，避免递归调用。

```swift
private func printInReverseNoRecursion<T>(_ list: LinkedList<T>) {
    var current = list.head
    var stack = Stack<T>()
    
    while let node = current {
        stack.push(node.value)
        current = node.next
    }
    
    while let value = stack.pop() {
        print(value)
    }
}
```

```swift
example(of: "Print Linkedlist reverse without recursion") {
    var list = LinkedList<Int>()
    list.push(3)
    list.push(2)
    list.push(1)
    
    print("Original list: \(list)")
    print("Printing in reverse: ")
    printInReverseNoRecursion(list)
}

/*
---Example of Print Linkedlist reverse without recursion---
Original list: 1 ->2 ->3  
Printing in reverse: 
3
2
1
*/
```


## Challenge 2：检查括号是否平衡。给定一个字符串，检查是否有 `（` 和 `）` 字符，如果字符串中的括号是平衡的，则返回 `true`。例如：

```txt
// 1 h((e))llo(world)() // balanced parentheses

// 2 (hello world // unbalanced parentheses
```

```swift
func checkParentheses(_ string: String) -> Bool {
    var stack = Stack<Character>()
    
    for character in string {
        if character == "(" {
            stack.push(character)
        } else if character == ")" {
            if stack.isEmpty {
                return false
            }else {
                stack.pop()
            }
        }
    }
    return stack.isEmpty
}
```

```swift
example(of: "check parentheses") {
    let string = "h((e))llo(world)())"
    
    let result = checkParentheses(string)
    
    print("\(result)")
}

/*
---Example of check parentheses---
false
*/
```