---
layout: post
author: Robin
title: \#10\ 二叉树及其有序、前序和后序遍历
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/10/cover.jpg'
---

在上一文中认识了一般树结构，其每一个节点可能会有多个子节点。二叉树也是树型结构，只不过其每一个节点最多只有两个节点，通常称为**左节点**和**右节点**。

![](/assets/Data-Structures-&-Algorithms-in-Swift/10/binary-tree.png)

## 二叉树的Swift实现

首先定义二叉树的基本属性，如下：

```swift
public class BinaryNode<Element> {
    public var value: Element
    public var leftChild: BinaryNode?
    public var rightChild: BinaryNode?
    
    public init(_ value: Element) {
        self.value = value
    }
}
```

有了二叉树的基本属性之后，就可以定义一颗二叉树了，如下：

```swift
var tree: BinaryNode<Int> = {
    let zero = BinaryNode(value: 0)
    let one = BinaryNode(value: 1)
    let five = BinaryNode(value: 5)
    let seven = BinaryNode(value: 7)
    let eight = BinaryNode(value: 8)
    let nine = BinaryNode(value: 9)
    
    seven.leftChild = one
    one.leftChild = zero
    one.rightChild = five
    seven.rightChild = nine
    nine.leftChild = eight
    
    return seven
}()
```

该二叉树的形态即如下图：

![](/assets/Data-Structures-&-Algorithms-in-Swift/10/binary-tree-demo.png)

## 二叉树图

数据结构的图像化能够帮助进一步理解数据结构，为了能够更加清晰的查看二叉树的树形结构，这里我们构造一个打印函数，以在Console中打印出二叉树的树形结构图。

```swift
extension BinaryNode: CustomStringConvertible {
    public var description: String {
        return diagram(for: self)
    }
    
    private func diagram(for node: BinaryNode?,
                         _ top: String = "",
                         _ root: String = "",
                         _ bottom: String = "") -> String {
        guard let node = node else {
            return root + "nil \n"
        }
        if node.leftChild == nil && node.rightChild == nil {
            return root + "\(node.value)\n"
        }
        return diagram(for: node.rightChild,
                       top + " ",
                       top + "┌──",
                       top + "│ ") + root + "\(node.value)\n" + diagram(for: node.leftChild,
                                                                          bottom + "│ ",
                                                                          bottom + "└──",
                                                                          bottom + " ")
    }
}
```

然后对前面构建的二叉树进行打印，输出如下：

```swift
---Example of tree diagram---
 ┌──nil 
┌──9
│ └──8
7
│ ┌──5
└──1
 └──0
```

## 遍历算法

在一般树中学习了深度优先和广度优先两种树型结构的遍历算法，经过小小的改动后，这两种遍历算法同样适用于二叉树的遍历。但是在二叉树结构中，更加关注的是另外三种遍历算法：有序、前序和后序遍历。

### 有序遍历

有序遍历按照如下的顺序遍历节点：

1. 如果当前节点有左节点，则递归访问该节点的子节点；
2. 然后访问当前节点
3. 如果当前节点有有节点，则继续递归遍历该节点的子节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/10/in-order.png)

算法实现如下：

```swift
extension BinaryNode {
    public func traverseInOrder(visit: (Element) -> Void) {
        leftChild?.traverseInOrder(visit: visit)
        visit(value)
        rightChild?.traverseInOrder(visit: visit)
    }
}
```

```swift
example(of: "in-order traversal") {
    tree.traverseInOrder {
        print($0)
    }
}

/*
---Example of in-order traversal---
0
1
5
7
8
9
*/
```

### 前序遍历

前序遍历总是先访问当前节点，然后递归遍历左节点和有节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/10/pre-order.png)

```swift
    public func traversePreOrder(visit: (Element) -> Void) {
        visit(value)
        leftChild?.traversePreOrder(visit: visit)
        rightChild?.traversePreOrder(visit: visit)
    }
```

```swift
example(of: "pre-order traversal") {
    tree.traversePreOrder {
        print($0)
    }
}
/*
---Example of ore-order traversal---
7
1
0
5
9
8
*/
```

### 后序遍历

后序遍历总是先递归访问当前节点的左节点和右节点，然后在访问当前节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/10/post-order.png)

```swift
    public func traversePostOrder(visit: (Element) -> Void) {
        leftChild?.traversePostOrder(visit: visit)
        rightChild?.traversePostOrder(visit: visit)
        visit(value)
    }
```

```swift
example(of: "post-order traversal") {
    tree.traversePostOrder {
        print($0)
    }
}
/*
---Example of post-order traversal---
0
5
1
8
9
7
*/
```

## 关键点总结

* 二叉树数据结构是其他树形数据结构的基础，很多类型的树例如二叉搜索树、AVL树等，都以二叉树为基础，并在其上增加插入和删除等操作行为；
* 有序、前序和后序并不仅仅针对二叉树非常重要，在其他的树形结构中，这些遍历算法依然有效且重要。