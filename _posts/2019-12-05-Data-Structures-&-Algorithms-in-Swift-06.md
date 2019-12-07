---
layout: post
author: Robin
title: \#6\ Linked List 挑战
tags: 数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/6/cover.jpg'
---

本文内容将针对LinkedList的五大通用性场景问题，进行求解。这些问题相比多数挑战来说相对简单，主要是为了巩固关于LinkedList的知识。

## Challenge 1：创建按照反向顺序打印链表元素的函数。

```txt
// LinkedList
1 -> 2 -> 3 -> nil

// outut
3
2
1
```

>  解决此问题最简单直接的方式就是使用**递归**。由于递归允许构建回调堆栈，因此我们可以在递归的回调中调用**print**打印节点元素值。

```swift
// 递归调用
private func printInReverse<T>(_ node: Node<T>?) {
    guard let node = node else {
        return
    }
    printInReverse(node.next)
    print(node.value)
}


func printInReverse<T>(_ list: LinkedList<T>) {
    printInReverse(list.head)
}
```
 
测试和结果检查：

```swift
example(of: "printing in reverse") {
    var list = LinkedList<Int>()
    list.push(3)
    list.push(2)
    list.push(1)
    
    print("Original list: \(list)")
    print("Printing in reverse: ")
    printInReverse(list)
}

// ---Example of printing in reverse---
// Original list: 1 ->2 ->3  
// Printing in reverse: 
// 3
// 2
// 1
```

该算法的核心在于递归调用的部分，当节点存在的情况下，继续遍历当前节点的下一个节点，否则就是已经到了末尾节点，在递归的过程回调堆栈中打印节点值。该算法时间复杂度为**O(n)**。

## Challenge 2：创建返回链表中间节点值的函数。


```txt
// LinkedList
1 -> 2 -> 3 -> 4 -> nil
// middle is 3

1 -> 2 -> 3 -> nil
// middle is 2
```

>  该问题的解决思路是利用**双指针位移的偏移量**的方式来进行求解，也就是说分别定义两个初始位置相同的指针，然后对链表进行遍历，遍历的过程中，其中一个针对每次位移两个位置，另一个位移一个位置，位移快的那个移动到链表末尾时，慢的那个正好是链表的中间位置。

```swift
func getMiddle<T>(_ list: LinkedList<T>) -> Node<T>? {
    var fast = list.head
    var slow = list.head
    
    while let nextFast = fast?.next {
        fast = nextFast.next
        slow = slow?.next
    }
    return slow
}
```


测试和结果检查：

```swift
example(of: "getting the middle node") {
    var list = LinkedList<Int>()
    list.push(3)
    list.push(2)
    list.push(1)
       
    print("Original list: \(list)")
    if let middleNode = getMiddle(list) {
        print(middleNode.value)
    }
}

// ---Example of getting the middle node---
// Original list: 1 ->2 ->3  
// 2
```

该算法的时间复杂度是**O(n)**。也可以使用另一种解法，先遍历依次整个链表，记录节点总数，然后取链表节点总数的一半，再次进行遍历，获取中间值，但是这样的解法需要遍历两次，时间复杂度为**O(n^2)**。

## Challenge 3：创建反转链表的函数。

```txt
// LinkedList
// Before
1 -> 2 -> 3 -> nil

// After
3 -> 2 -> 1 -> nil
```

> 该问题简单的解决方案是，新建一个LinkedList，然后遍历原LinkedList，将节点一个一个的push到新的LinkedList，最后更新原LinkedList的头节点即可。但是这样的方式会有一个性能问题，就是每次调用push方法的时候，都需要分配新的节点，造成了绝大的资源成本。另一种代码较为复杂，但是性能上却相当好的方案是，构建两个变量，分别指向当前节点和上一个节点，然后遍历LinkedList，依次向后交换当前节点和上一个节点的指向，直到当前节点为nil时结束，这样就完全避免了每次新建节点的资源消耗问题。

```swift
// Reverse solution 1
mutating func reverseSolutionOne() {
    var tempList = LinkedList<Value>()
    for value in self {
        tempList.push(value)
    }
    head = tempList.head
}
```

```swift
// Reverse solution 2
public mutating func reverseSolutionTwo() {
    tail = head
    var prev = head
    var current = head?.next
    prev?.next = nil
        
    while current != nil {
        let next = current?.next
        current?.next = prev
        prev = current
        current = next
    }
    head = prev
}
```

虽然两种解决方案都是完整该挑战，但是在时间复杂度相同的情况下，空间复杂度更好的解决方案2，是应该遵循且掌握的方式。算法2的思路图示如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/6/reversed-list.png)


测试算法及检验结果：

```swift
example(of: "reverse the list solution 2") {
    var list = LinkedList<Int>()
    list.push(3)
    list.push(2)
    list.push(1)
    
    print("Original list: \(list)")
    list.reverseSolutionTwo()
    print("Reversed list: \(list)")
}

// ---Example of reverse the list solution 2---
// Original list: 1 ->2 ->3  
// Reversed list: 3 ->2 ->1 
```

## Challenge 4：创建一个函数，该函数接收两个已排序的链表，并合并到单个排序的链表中。

```txt
// list1
1 -> 4 -> 10 -> 11

// list2
-1 -> 2 -> 3 -> 6

// merged list
-1 -> 1 -> 2 -> 3 -> 4 -> 6 -> 10 -> 11
```

> 此问题的解决方案是不断从两个已排序的列表中摘取节点，并将它们添加到新列表中。由于两个列表已经排序，因此可以比较两个列表的下一个节点，以查看哪个节点应该是要添加到新列表的下一个节点。

```swift
func mergeSort<T: Comparable>(_ left: LinkedList<T>, _ right:LinkedList<T>) -> LinkedList<T> {
    // 检查输入的两个链表是否为空，如果其中一个为空，则直接返回另一个
    guard !left.isEmpty else {
        return right
    }
    guard !right.isEmpty else {
        return left
    }
    
    // 结果链表的head、tail定义
    var newHead: Node<T>?
    var tail: Node<T>?
    
    var currentLeft = left.head
    var currentRight = right.head
    // 检查left、right的首个节点，并将小的节点赋值给newHead
    if let leftNode = currentLeft, let rightNode = currentRight {
        if leftNode.value < rightNode.value {
            newHead = leftNode
            currentLeft = leftNode.next
        } else {
            newHead = rightNode
            currentRight = rightNode.next
        }
        tail = newHead
    }
    // 合并
    // 遍历left、right，尝试挑选能够加入新链表的节点，直到其中一个链表到达末尾节点
    while let leftNode = currentLeft, let rightNode = currentRight {
        // 比较节点值大小，并将小的链接到tail.next
        if leftNode.value < rightNode.value {
            tail?.next = leftNode
            currentLeft = leftNode.next
        } else {
            tail?.next = rightNode
            currentRight = rightNode.next
        }
        tail = tail?.next
    }
    // 上个while循环同时以来currentLeft和currentRight，因此即使链表中还有节点，循坏也可能提前终止。需要将剩余的节点链接到处理单元中
    if let leftNodes = currentLeft {
        tail?.next = leftNodes
    }
    
    if let rightNodes = currentRight {
        tail?.next = rightNodes
    }
    
    // 创建结果链表，这里不使用push或者append的方式，而是直接指定链表的head、tail
    // head只有一个节点，直接复制，tail包含了很多节点，需要一个一个地进行链接
    var list = LinkedList<T>()
    list.head = newHead
    list.tail = {
        while let next = tail?.next {
            tail = next
        }
        return tail
    }()
    return list
}
```

算法求解过程的图示：

![](/assets/Data-Structures-&-Algorithms-in-Swift/6/MergeTwolinkedLists.png)

```swift
example(of: "merging two sorted list") {
    var list1 = LinkedList<Int>()
    list1.push(3)
    list1.push(2)
    list1.push(1)
    
    var list2 = LinkedList<Int>()
    list2.push(-1)
    list2.push(-2)
    list2.push(-3)
    
    print("First list: \(list1)")
    print("Second list: \(list2)")
    let mergedList = mergeSort(list1, list2)
    print("Merged list: \(mergedList)")
}

// ---Example of merging two sorted list---
// First list: 1 ->2 ->3  
// Second list: -3 ->-2 ->-1  
// Merged list: -3 ->-2 ->-1 ->1 ->2 ->3 
```

## Challenge 5：创建从链表中删除特定元素的所有匹配项的函数。

```txt
// original list
1 -> 3 -> 3 -> 3 -> 4

// list after removing all occurrences of 3
1 -> 4
```

```swift
extension LinkedList where Value: Equatable {
    public mutating func removeAll(_ value: Value) {
        while let head = self.head, head.value == value {
            self.head = head.next
        }
        
        var prev = head
        var current = head?.next
        while let currentNode = current {
            guard currentNode.value != value else {
                prev?.next = currentNode.next
                current = prev?.next
                continue
            }
            prev = current
            current = current?.next
        }
        
        tail = prev
    }
}
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/6/delete-duplicate.png)

测试算法及检验结果：

```swift
example(of: "deleting duplicate nodes") {
    var list1 = LinkedList<Int>()
    list1.push(3)
    list1.push(2)
    list1.push(2)
    list1.push(2)
    list1.push(1)
    list1.push(1)
    
    print("Origin list: \(list1)")
    list1.removeAll(2)
    print("Delete duplicate list: \(list1)")
}

// ---Example of deleting duplicate nodes---
// Origin list: 1 ->1 ->2 ->2 ->2 ->3     
// Delete duplicate list: 1 ->1 ->3 
```

