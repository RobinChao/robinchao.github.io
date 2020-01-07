---
layout: post
author: Robin
title: \#5\ Linked List && Swift Collection Protocol
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/5/cover.jpg'
---

在Swift标准库（**Swift standard library**）中定义了很多协议或协议的集合，这些协议分别对应了特定的数据类型，每个协议都对所定义的数据类型有一些特性和性能方面的保证，而对于开发者而言，这些协议也是自定义数据结构和对现有数据类型进行扩展的基础准则。在这些协议的集合中，有四种关于**集合的协议（collection protocols）**，分别是：

* **Tier 1, Sequence：**序列类型是Swift中最为朴素的协议,仅仅定义了一系列类型相同的元素，而不对这一系列元素的性质有任何额外的约定。它唯一约定了的动作，就是从序列当前位置读取下一个元素。
* **Tier 2, Collection：**集合类型是一种提供额外保证的序列类型。集合类型是有限的，允许重复的非破坏性顺序访问。
* **Tier 3, BidirectionalColllection：**集合类型可以是双向集合类型，可以允许在序列中上下双向移动。 这对于链表是不可能的，因为你只能从头到尾，而不是相反。
* **Tier 4, RandomAccessCollection：**如果它可以保证访问特定索引处的元素将花费与访问任何其他索引处的元素一样长的时间。该双向集合类型就是随机访问集合类型， 这对于链表来说是不可能的，因为访问列表前面附近的节点比列表下方的节点快得多。

因此对于链表数据结构来说，**Sequence**和**Collection**两种协议是适用的。首先链表是一个序列型数据结构，适用**Sequence**协议，另外链表是有限序列，适用**Collection**协议。

## 进化为Swift集合

集合类型是有限序列，并提供非破坏性顺序访问。 Swift Collection还允许通过**下标（subscript）**进行访问, 使用索引可以映射到集合中的值。

例如Swift中Array通过下标的方式访问元素：

```swift
array[5]
```

数组的下标一律是整数类型，例如上例中5。下标被包裹在方括号内。通过下标可以获取到集合对应未知的元素。

### 自定义集合索引

衡量**Collection**协议性能的指标是下标对应到值的速度。和其他数据类型（例如Swift的Array）不同，链表结构不能使用整数实现O(1)的下标操作。因此，自定义下标索引是对各自节点引用的索引。

在上文[\#4\ Linked List 的Swift实现](https://robinchao.github.io/2019/12/03/Data-Structures-&-Algorithms-in-Swift-04.html)的**LinkedList.swift**中，继续添加如下扩展程序，实现自定义索引的操作：

```swift
extension LinkedList: Collection {
    // 自定义链表索引
    // 由于索引是一个可比较的对象，需要继承Comparable协议
    public struct Index: Comparable {
        public var node: Node<Value>?
        
        // 自定义结构体不能进行==操作, 需要自行实现
        static public func ==(lhs: Index, rhs: Index) -> Bool {
            // 属于switch语句中使用元组
            switch (lhs.node, rhs.node) {
            case let (left?, right?):
                return left.next === right.next
            case (nil, nil):
                return true
            default:
                return false
            }
        }
        // 第一个参数是否小于第二个参数
        static public func <(lhs: Index, rhs: Index) -> Bool {
            guard lhs != rhs else {
                return false
            }
            // 从链表的一个节点移动到根节点
            // 这里使用Swift的内联序列函数sequence(first: next:)，类似repeat...while操作
            let nodes = sequence(first: lhs.node) { 
                $0?.next 
            }

            return nodes.contains { 
                $0 === rhs.node 
            }
        }
    }
    
    // 链表的头节点索引
    public var startIndex: Index {
        return Index(node: head)
    }
    // 链表的尾节点索引
    // 由于Collection协议的endIndex默认是序列最后可访问的值的索引，对于链表来说，需要制定tail节点的next
    public var endIndex: Index {
        return Index(node: tail?.next)
    }
    // 索引是可以递增的，给定索引的下一个索引就是当前节点的next
    public func index(after i: Index) -> Index {
        return Index(node: i.node?.next)
    }
    // 用于将索引映射到集合中的值。由于已经创建了自定义索引，因此可以通过引用节点的值在恒定时间内轻松实现此目的。
    public subscript(position: Index) -> Value {
        return position.node!.value
    }
}
```

回到主Playground，编写自定义索引功能的使用操作，如下：

```swift
example(of: "using collection") {
    var list = LinkedList<Int>()
    
    for i in 0 ... 9 {
        list.append(i)
    }
    
    print("List: \(list)")
    print("First element: \(list[list.startIndex])")
    print("Array containing first 3 elements: \(Array(list.prefix(3)))")
    print("Array containing last 3 elements: \(Array(list.suffix(3)))")
    
    let sum = list.reduce(0, +)
    print("Sum of all values: \(sum)")
}

// ---Example of using collection---
// List: 0 ->1 ->2 ->3 ->4 ->5 ->6 ->7 ->8 ->9         
// First element: 0
// Array containing first 3 elements: [0, 1, 2]
// Array containing last 3 elements: [7, 8, 9]
// Sum of all values: 45
```

### 值语义和写入时复制（copy-on-write）

Swift Collection的另一个重要特性是它们具有值语义，通过**写入时复制**实现的，特此称为 **COW**。为了说明此概念，您将使用数组验证此行为。在Playground页面的底部编写以下内容：

```swift
example(of: "array cow") {
    let array1 = [1, 2]
    var array2 = array1
    
    print("array1: \(array1)")
    print("array2: \(array2)")
    
    print("--- After adding 3 to array 2 ---")
    array2.append(3)
    print("array1: \(array1)")
    print("array2: \(array2)")
}

// ---Example of array cow---
// array1: [1, 2]
// array2: [1, 2]
// --- After adding 3 to array 2 ---
// array1: [1, 2]
// array2: [1, 2, 3]
```
array1是不可变的常量，就算将array1赋值给了变量array2，当array2的内容改变的时候，array1也不会改变，而此时有一个关键的地方是，array2在被赋值为array1的时候，并没有开辟新的存储空间，而是指向了array1的存储空间。当对array2进行append操作时，array2才对array1的内存空间进行了一个拷贝，然后添加了元素3。

![](/assets/Data-Structures-&-Algorithms-in-Swift/5/array-cow.png)

了解了值语义之后，来检查我们首先的Linked List是否也具有值语义的特性，在Playground中变下如下测试代码：

```swift
example(of: "linked list cow test") {
    var list1 = LinkedList<Int>()
    list1.append(1)
    list1.append(2)
    
    var list2 = list1
    
    print("list1: \(list1)")
    print("list2: \(list2)")
    
    print("--- After adding 3 to list 2 ---")
    list2.append(3)
    print("list1: \(list1)")
    print("list2: \(list2)")
}

// ---Example of linked list cow test---
// list1: 1 ->2 
// list2: 1 ->2 
// --- After adding 3 to list 2 ---
// list1: 1 ->2 ->3  
// list2: 1 ->2 ->3
```

可以看到，我们实现的LinkedList并不具有值语义的特性。因为我们在基础存储的时候使用了引用类型（Node），在Swift中，结构体应该是支持值语义的，因此关于LinkedList的实现，还需要进行优化，以支持值语义特性。

使用**COW**实现值语义的特性相对较为简单。在更改链表的内容之前，需要对基础存储部分进行**copy**操作，同时将链表的所有引用（head、tail）更新到新的copy副本中。实现代码如下：

```swift
private mutating func copyNodes() {
    guard var oldNode = head else {
        return
    }
        
    head = Node(value: oldNode.value)
    var newNode = head
        
    while let nextOfNode = oldNode.next {
        newNode!.next = Node(value: nextOfNode.value)
        newNode = newNode!.next
            
        oldNode = nextOfNode
    }
    tail = newNode
}
```

该操作是原有链表节点的值赋值给新建的节点，为链表的所有节点建立了一个新的副本。接下来需要修改LinkedList中的一些方法，增加`copyNodes()`方法的调用，以支持值语义特性。

* push
* append
* insert(after:)
* pop
* removeLast
* remove(after:)

完整了上述方法的修改后，回到主Playground，进行值语义特性的再次测试，得到如下结果：

```swift
// ---Example of linked list cow test---
// list1: 1 ->2 
// list2: 1 ->2 
// --- After adding 3 to list 2 ---
// list1: 1 ->2 
// list2: 1 ->2 ->3 
```

得到的结果也符合值语义的特性，但是这里有一个问题，在加入了值语义特性后，在每一个支持**mutating**的方法中，都多了一个**O(n)**的copy操作，显得得不偿失。

## COW的优化

每一个支持**mutating**的方法中，都多了一个**O(n)**的copy操作，显然是不可接受的。接下来着手对其进行进一步的优化，有两种方式可以帮助解决这个问题。第一种便是当节点仅有一个拥有者的时候，避免进行复制。

### isKnownUniquelyReferenced

在Swift的标准库中,有一个函数**isKnownUniquelyReferenced**,该函数可用于检查对象是否只有一个引用。使用该函数对上述实现进行测试，在上述值语义的测试代码中的`var list2 = list1`语句前后，添加检查：

```swift
print("List1 uniquely referenced: \(isKnownUniquelyReferenced(&list1.head))")
var list2 = list1
print("List1 uniquely referenced: \(isKnownUniquelyReferenced(&list1.head))")
```

执行后，打印结果如下：

```sh
List1 uniquely referenced: true
List1 uniquely referenced: false
```

使用**isKnownUniquelyReferenced**能够检查node对象是否被共享。验证了此函数的功能后，删除上述打印语句，在**copyNodes()**函数中添加检查性代码：

```swift
guard !isKnownUniquelyReferenced(&head) else {
    return
}
```

添加后，测试COW，打印结果：

```swift
// ---Example of linked list cow test---
// list1: 1 ->2 
// list2: 1 ->2 
// --- After adding 3 to list 2 ---
// list1: 1 ->2 
// list2: 1 ->2 ->3 
```

可以看到值语义特性依然运行良好。通过这个优化，LinkedList的性能将借助COW的特性恢复到之前的水平。

### 节点共享

> 节点共享是在禁用COW的情况下的另一种方式，因此在下面的工作原理中，均属于禁用COW的范畴。

另一种优化COW的方式是通过**节点部分共享**的方式。在一些情况下，并不需要完全复制整个链表，其中部分节点可以采用共享的方式实现。其工作原理如下：

```swift
example(of: "share nodes ") {
    var list1 = LinkedList<Int>()
    (1 ... 3).forEach { list1.append($0) }
    var list2 = list1
}
```

上述代码中`list2`并未新建内存空间，而是将指针指向了list1对应的位置。

![](/assets/Data-Structures-&-Algorithms-in-Swift/5/share-nodes-eg1.png)

接下来向list2中添加元素：

```swift
list2.push(0)
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/5/share-nodes-eg2.png)

通过上述图例可知，list1和list2两个链表共享了节点1、2、3，并且list1的头节点属于共享节点1。

```sh
list1: 1 ->2 ->3  
list2: 0 ->1 ->2 ->3 
```

如果此时向list1添加元素：

```swift
list1.push(100)
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/5/share-nodes-eg3.png)

打印结果如下：

```sh
list1: 100 ->1 ->2 ->3   
list2: 0 ->1 ->2 ->3  
```

可以看到两个链表依然共享节点1、2、3，list1的头节点已经改变。

**节点共享**的方式是另一种可以实现类似COW特性的方式，在值语义的功能中可能会比Copy操作更加有效。这里不再进行具体的实现。

## 关键点总结

* **单向链表是一个线性的、单向的数据结构**，一旦将引用从一个节点移动到另一个节点，将无法返回；
* 链接列表具有头插入的 **O（1）** 时间复杂性。数组具有 **O（n）** 时间复杂性；
* 符合 Swift 集合协议（如**Sequence**和**Collection**）为相当少量的需求提供了大量有用的方法；
* 通过写**入时复制（COW）**行为，您可以实现值语义。