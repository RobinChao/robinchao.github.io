---
layout: post
author: Robin
title: \#12\ 自平衡二叉搜索树（AVL Trees）
tags: 数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/11/cover.jpg'
---

在上文中，已经了解二叉搜索树的O(log n)性能特征，但是当二叉搜索树节点删除中，可能会出现不平衡的树，并降低树的性能到O(n)。这一文的内容将学习另一种改进了的二叉搜索树 --- 自平衡二叉搜索树。

> 1962年，Georgy Adelson-Velsky和Evgenii Landis提出了第一个自平衡二进制搜索树：AVL树。

## 理解什么是平衡

自平衡二叉搜索树，简称平衡树，是在二叉搜索树的基础上改进的，首先了解一下平衡树的三种平衡状态。

**完全平衡树**

二叉搜索树的理想形式便是完全平衡状态，也就是说二叉搜索树的每个层级从上之下均存在节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/perfect-tree.png)

完全平衡树不仅仅整棵树是对称的，而且最底部的叶子都均存在节点。

**“够好”的平衡树**

尽管完全平衡是理想的平衡树状态，但是在现实中往往难以实现，要达到完全平衡的树结构，需要节点有确切的数量，才有可能达到完全平衡，因此只有一定数量的元素才能平衡。

例如一棵树有1个、3个、4个节点的时候，是可以达到完全平衡的，但是当树有2个、4个、5个或6个节点的时候，却不能达到完全平衡，因为树的最后一层无法完全被填满。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/balance-tree.png)

平衡树的定义是树的每一个层级必须被填满（底部层级除外）。在大多数情况下，二叉树是可以做到的最好的结构。

**不平衡树**

最后一种状态是不平衡的状态，二叉搜索树的不平衡状态会带来各种等级的性能损失。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/unbanlance-tree.png)

保持树的平衡会带来查找、插入和移除元素操作O(log n)的时间复杂度。当树的结构不平衡是，AVL树会自动调整树的结构，使得树结构保持平衡，从而带来良好地时间复杂度等。

## 实现

AVL树和二叉搜索树有很多地方都是相同的实现，但是为了便于区分，我们将新建文件重新编写关于AVL树的实现。

### 平衡度衡量

要确保二叉树的平衡，就需要一种能够衡量二叉树是否平衡的方法。在AVL树中，每个节点都有一个 **height** 属性，该属性描述了当前节点到叶子节点的**最长距离**。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/node-marks-height.png)

我们新建AVLNode类，添加存储节点高度的变量：

```swift
public var height = 0
```

在使用中，将使用节点的相对高度来确定节点是否平衡。

AVL树中左节点和有节点的高度差最多相差1，这个差值称之为**平衡因子**。为了计算的方便，对节点增加如下变量：

```swift
public var balanceFactor: Int {
    return leftHeight - rightHeight
}
    
public var leftHeight: Int {
    return leftChild?.height ?? -1
}
    
public var rightHeight: Int {
    return rightChild?.height ?? -1
}
```

`balanceFactor`平衡因子计算的是左树和右树的高度差，如果子节点为nil，则节点的高度为-1。例如：

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/avl-tree-demo.png)

该树是一个平衡的二叉树，但是如果增加一个节点40的话，就会改变为不平衡的树：

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/unbalance-tree.png)

从图中的平衡因子即可判断出该树处于一个不平衡的状态。为了使得不平衡树改变形态，而成为平衡树，需要进一步的操作。


### 旋转

应用于二叉搜索树，使得其平衡的操作称之为**旋转**。共有四种选装的操作，分别是**左旋转**、**左-右旋转**、**右旋转**、**右-左旋转**。

**左旋转**

例如上述不平衡的二叉树中，插入节点40导致树处于不平衡的状态，而是用左旋转可以解决此不平衡问题。左旋转的工作原理如下图：

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/left-rotation.png)

树在旋转前后会由两点不同之处：

* 树节点的有序遍历保持不变；
* 旋转后，树的深度减少一级。

```swift
private func leftRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    let pivot = node.rightChild!
    node.rightChild = pivot.leftChild
    pivot.leftChild = node
    
    node.height = max(node.leftHeight, node.rightHeight) + 1
    pivot.height = max(pivot.leftHeight, pivot.rightHeight) + 1
    
    return pivot
}
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/left-rotation-done.png)

**右旋转**

右旋转和左旋转类似，如果是由于左节点导致了树不平衡，则使用右旋转的方式，其原理如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/right-rotation.png)

```swift
private func rightRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    let pivot = node.leftChild!
    node.leftChild = pivot.rightChild
    pivot.rightChild = node
        
    node.height = max(node.leftHeight, node.rightHeight) + 1
    pivot.height = max(pivot.leftHeight, pivot.rightHeight) + 1
    
    return pivot
}
```

**右-左旋转**

左旋转和右旋转的操作都有一个特点是：均平衡了树的左节点或右节点。假设有如下的不平衡树：

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/left-right-nodes-tree.png)

单纯使用左旋转不能得到平衡树，针对此种情况可以先进行右旋转，使得节点均有右节点，然后再使用左旋转，以达到平衡树状态。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/right-left-rotation.png)

```swift
private func rightLeftRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    guard let rightChild = node.rightChild else {
        return node
    }
    
    node.rightChild = rightRotate(rightChild)
    return leftRotate(node)
}
```

**左-右旋转**

左-右旋转和右-左旋转针对的情形类似，当单纯使用左旋转无法达到平衡的时候，根据节点的情况，再次进行右旋转，以达到平衡状态。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/left-right-rotation.png)

```swift
private func leftRightRotate(_ node: AVLNode<Element>) -> AVLNode<Element> {
    guard let leftChild = node.leftChild else {
        return node
    }
    
    node.leftChild = leftRotate(leftChild)
    return rightRotate(node)
}
```

### 平衡

进行了树的节点旋转后，虽然形态上会达到平衡，但是某些节点可能破坏了二叉搜索树的特性，因此还需要根据不同的情形，对数的节点进行交换，这里交换的根据是根据节点的平衡因子进行不同的操作。算法结构如下：

```swift
private func balanced(_ node: AVLNode<Element>) -> AVLNode<Element> {
  switch node.balanceFactor {
  case 2:
    // ...
  case -2:
    // ...
  default:
    return node
  }
}
```

该算法会根据三种情况进行节点的转换：

1. 平衡因子*balanceFactor*为2的时候，表示节点的左子节点多于右子节点，意味着节需要使用右旋或者左-右旋转；
2. 平衡因子*balanceFactor*为-2的时候，表示节点的右子节点多于左子节点，意味着需要使用左旋或者右-左旋转；
3. 除了这两种情况之外，表示当前节点处于平衡状态，不需要进行任何的转换。

平衡因子*balanceFactor*符号能够确定需要使用单旋还是双旋。

![](/assets/Data-Structures-&-Algorithms-in-Swift/12/balanceFactor-single-or-double-rotation.png)

完整的算法如下：

```swift
private func balanced(_ node: AVLNode<Element>) -> AVLNode<Element> {
    switch node.balanceFactor {
    case 2:
        if let leftChild = node.leftChild, leftChild.balanceFactor == -1 {
            return leftRightRotate(node)
        } else {
            return rightRotate(node)
        }
    case -2:
        if let rightChild = node.rightChild, rightChild.balanceFactor == 1 {
            return rightLeftRotate(node)
        } else {
            return leftRotate(node)
        }
    default:
        return node
    }
}
```

### 节点的插入

至此，节点调整，使得树平衡的工作已经完成了，而该平衡操作大部分情况下是在向树插入新的节点的时候进行的，因此完成插入算法如下：

```swift
public mutating func insert(_ value: Element) {
    root = insert(from: root, value: value)
}
    
private func insert(from node: AVLNode<Element>?, value: Element) -> AVLNode<Element> {
    guard let node = node else {
        return AVLNode(value: value)
    }
    if value < node.value {
        node.leftChild = insert(from: node.leftChild, value: value)
    } else {
        node.rightChild = insert(from: node.rightChild, value: value)
    }
        
    let balancedNode = balanced(node)
    balancedNode.height = max(balancedNode.leftHeight, balancedNode.rightHeight) + 1
    return balancedNode
}
```

节点的插入算法也适用于新建一颗AVL树，例如：

```swift
example(of: "repeated insertions in sequence") {
    var tree = AVLTree<Int>()
    for i in 0 ..< 15 {
        tree.insert(i)
    }
    print(tree)
}

/*
---Example of repeated insertions in sequence---
  ┌──14
 ┌──13
 │ └──12
┌──11
│ │ ┌──10
│ └──9
│  └──8
7
│  ┌──6
│ ┌──5
│ │ └──4
└──3
 │ ┌──2
 └──1
  └──0
*/
```

### 节点的移除

移除树中的节点，也会导致树不平衡，因此在移除算法中也需要对树的平衡性进行调整。

```swift
public mutating func remove(_ value: Element) {
    root = remove(from: root, value: value)
}

private func remove(from node: AVLNode<Element>?, value: Element) -> AVLNode<Element>? {
    guard let node = node else {
        return nil
    }
        
    if value == node.value {
        if node.leftChild == nil && node.rightChild == nil {
            return nil
        }
        if node.leftChild == nil {
            return node.rightChild
        }
        if node.rightChild == nil {
            return node.leftChild
        }
        node.value = node.rightChild!.min.value
        node.rightChild = remove(from: node.rightChild, value: node.value)
    } else if value < node.value {
        node.leftChild = remove(from: node.leftChild, value: value)
    } else {
        node.rightChild = remove(from: node.rightChild, value: value)
    }
        
    let balancedNode = balanced(node)
    balancedNode.height = max(balancedNode.leftHeight, balancedNode.rightHeight) + 1
    return balancedNode
}
```

测试如下：

```swift
example(of: "removing a vaue") {
    var tree = AVLTree<Int>()
    tree.insert(15)
    tree.insert(10)
    tree.insert(16)
    tree.insert(18)
    print(tree)
    
    tree.remove(10)
    print(tree)
}

/*
---Example of removing a vaue---
 ┌──18
┌──16
│ └──nil 
15
└──10

┌──18
16
└──15
*/
```

移除了元素10之后，树进行了平衡调整（左旋），依然保持树的平衡性。由于节点的插入和移除操作同于二叉搜索树，因此其时间复杂度依然是O(log n)。

## 关键点总结

* 自平衡树在节点的插入和移除过程中，增加平衡性调节操作，保持其执行效率不会降低；
* AVL树在树不平衡的情况下，通过重新调整树的部分节点，而使得树保持平衡。