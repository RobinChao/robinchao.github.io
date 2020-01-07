---
layout: post
author: Robin
title: \#11\ 二叉搜索树
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/11/cover.jpg'
---

二叉搜索树又称为二叉查找树（BST），是一种支持快速查找、插入和删除操作的树结构，例如下方的决策树，其中选择一方而放弃另一方的所有可能性，从而将问题减半。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/bst-decision-tree-demo.png)

在决策树中，一旦做出了决定并选择了某个分支，便不能回头，在选择的分支上一直查找直到叶子节点，得到最终决定。二叉搜索树在上一文中的二叉树的基础上增加了以下两个规则：

1. 左节点的值必须小于父节点的值；
2. 同样的，右节点的值必须大于等于其父节点的值。

二叉搜索树使用这两个规则可避免执行不必要的检查，其查找、插入和删除的平均时间复杂度为O(log n)，比线性结构快的多。

## Array vs. BST

例如有如下的两种结构的集合：

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/array-vs-bst.png)

上面的结构为数组，下方的为二叉搜索树结构。

### 搜索

对于未排序的数组来说，搜索只能从开头到结尾，例如搜索元素105。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/array-search-105.png)

这也是为什么*array.contains(:)*操作为O(n)的原因。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/bst-search-105.png)

使用二叉搜索树的数据结构，搜索同样的元素105，简单且高效的多。因为搜索算法每次访问节点的时候，都会遵循如下两个原则：

1. 如果目标搜索值小于当前节点的值，那么目标搜索值一定在当前节点的左树中；
2. 如果目标搜索值大于当前节点的值，那么目标搜索值一定在当前节点的右树中。

通过BST的这些搜索原则，可以避免一些数据项检查，减半搜索空间，这也是BST中搜索算法的时间复杂度为O(log n)的原因。

### 插入

对于数组来说，如果要插入一个元素，需要将插入位置之后的所有元素向后移动一个位置，这也是为什么对于数组的插入算法，时间复杂度为O(n)的原因。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/array-insertion.png)

而对于二叉搜索树来说，插入一个新的元素，相对舒服的多。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/bst-insertion.png)

由于二叉搜索树的规则，小于当前节点元素值的节点一定在当前节点的左树或左节点，因此插入一个新的元素的时候，可以减少一半的检查时间。例如上图中，在现有的二叉搜索树中插入新的元素1。二叉搜索树的插入算法，时间复杂度应该为O(log n)。

### 移除

和插入类似，数组的元素移除同样需要移动删除目标元素之后的所有元素的位置，因此其时间复杂度依然是O(n)。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/array-removal.png)

对于二叉搜索树来说，会由两种情况，当删除的节点是叶子节点的时候，相对较为简单，但是如果待删除的节点有子节点的时候，相对要进行一些复杂的管理工作。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/bst-removal.png)

## 结构与算法实现

```swift
public struct BinarySearchTree<Element: Comparable> {
    public private(set) var root: BinaryNode<Element>?
    public init() {}
}

extension BinarySearchTree: CustomStringConvertible {
    public var description: String {
        guard let root = root else {
            return "Empty tree"
        }
        return String(describing: root)
    }
}
```

二叉搜索树的本质是二叉树，因此在二叉搜索树的内部，定义的节点类型为BinaryNode，并且二叉搜索树中的元素时可比较的。

### 插入算法

二叉树的基本准则是：

1. 左节点的值必须小于父节点的值；
2. 同样的，右节点的值必须大于等于其父节点的值。

依据这两个准则，即可实现二叉搜索树的插入算法：

```swift
extension BinarySearchTree {
    public mutating func insert(_ value: Element) {
        root = insert(from: root, value: value)
    }
    
    private func insert(from node: BinaryNode<Element>?, value: Element) -> BinaryNode<Element> {
        // 如果是空的树，则直接使用当前值构建一个节点返回
        guard let node = node else {
            return BinaryNode(value: value)
        }
        // 插入到左树
        if value < node.value {
            node.leftChild = insert(from: node.leftChild, value: value)
        } else {
            // 插入到右树
            node.rightChild = insert(from: node.rightChild, value: value)
        }
        return node
    }
}
```

```swift
example(of: "building a BST") {
    var bst = BinarySearchTree<Int>()
    for i in 0 ..< 5 {
        bst.insert(i)
    }
    print(bst)
}

/*
---Example of building a BST---
   ┌──4
  ┌──3
  │ └──nil 
 ┌──2
 │ └──nil 
┌──1
│ └──nil 
0
└──nil 
*/
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/unbalanced-bst.png)

然而，对于上述构建的BST，貌似是不平衡的，二叉树的节点均在根节点的右树上，而理想的状态应该是上图中右侧的部分。如果此时插入新的元素5，则会出现如下情况：

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/unbalanced-bst-insertion.png)

> 解决方案是构建自平衡树的结构，自平衡树这里不进行详述。

为了防止树结构的不平衡，这里仅仅使用最为笨重的方式构建一个平衡的二叉树。

```swift
var exampleTree: BinarySearchTree<Int> {
    var bst = BinarySearchTree<Int>()
    bst.insert(3)
    bst.insert(1)
    bst.insert(4)
    bst.insert(0)
    bst.insert(2)
    bst.insert(5)
    return bst
}
```

```swift
example(of: "building a balanced BST") {
    print(exampleTree)
}

/*
---Example of building a balanced BST---
 ┌──5
┌──4
│ └──nil 
3
│ ┌──2
└──1
 └──0
*/
```

### 元素搜索

二叉搜索树元素的搜索需要遍历二叉树的各个节点，因此可以使用上一文中实现的二叉树相关遍历算法 --- 有序、前序和后序。

```swift
extension BinarySearchTree {
    public func contains(_ value: Element) -> Bool {
        guard let root = root else {
            return false
        }
        var found = false
        root.traverseInOrder {
            if $0 == value {
                found = true
            }
        }
        return found
    }
}
```

例如搜索上述二叉搜索树中是否包含元素5：

```swift
example(of: "finding a node") {
    if exampleTree.contains(5) {
        print("Found 5!")
    } else {
      print("Couldn't find 5")
    }
}

/*
---Example of finding a node---
Found 5!
*/
```

这里的元素搜索算法使用了有序遍历的方法，因此其时间复杂度也为O(n)。但是该复杂度的搜索算法存在可优化的空间，由于二叉搜索树有左节点小于父节点，右节点大于父节点的特性，因此在元素搜索的时候，可以利用该属性减少搜索检查的范围，进一步优化搜索耗时等。

优化元素的搜索算法如下：

```swift
    public func containsOpt(_ value: Element) -> Bool {
        var current = root
        while let node = current {
            if node.value == value {
                return true
            }
            
            if value < node.value {
                current = node.leftChild
            } else {
                current = node.rightChild
            }
        }
        return false
    }
```

优化的算法时间复杂度为O(log n)。

### 移除元素

二叉搜索树元素的移除相对元素的搜索和插入复杂一些，有如下三种场景需要考虑：

**Case 1：叶子节点**

对于待删除的元素所在的节点为叶子节点时，是最为简单直接的场景，直接分类该节点和其父节点的链接即可。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/remove-element-leaf-node.png)

**Case 2：有一个子节点的节点**

当待删除的元素所在的节点拥有一个子节点的时候，不仅仅要删除该节点，还需要将该节点的字节点和树的其余节点进行重新链接，一般情况下，会重新和删除节点的原始父节点进行链接。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/remove-element-one-node.png)

**Case 3：有两个子节点的节点**

当待删除节点拥有两个子节点的时候，删除操作相对较为复杂一点。例如下图二叉搜索树，想要删除的元素为25：

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/remove-element-two-node-1.png)

如果只是简单的删除节点25，如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/remove-element-two-node-2.png)

此时虽然删除掉了节点25，但是带来了另一个问题。当删除掉节点25之后，会出现两个需要重建链接的节点12和37，此时原节点25的父节点却只有一个子节点的空间，如果如上图那样建立链接的话，会导致该二叉搜索树不成立。为了解决此问题，需要对节点的链接进行交换，使得删除节点后的二叉搜索树依然成立。

方法就是，删除拥有两个节点的节点之后，使用其右侧子树中最小的节点替换删除的节点，根据二叉搜索树的特点，最小的节点即为右侧子树中最左的节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/11/remove-element-two-node-3.png)

这样进行节点交换之后，二叉搜索树依然有效，因为新节点是右子树中的最小节点，所以右子树中的所有节点仍将大于或等于新节点。并且由于新节点来自右子树，因此左子树中的所有节点都小于新节点。

**算法实现**

```swift
private extension BinaryNode {
    var min: BinaryNode {
        return leftChild?.min ?? self
    }
}

extension BinarySearchTree {
    public mutating func remove(_ value: Element) {
        root = remove(node: root, value: value)
    }
    
    private func remove(node: BinaryNode<Element>?, value: Element) -> BinaryNode<Element>? {
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
            node.rightChild = remove(node: node.rightChild, value: node.value)
        } else if value < node.value {
            node.leftChild = remove(node: node.leftChild, value: value)
        } else {
            node.rightChild = remove(node: node.rightChild, value: value)
        }
        return node
    }
}
```

```swift
example(of: "removing a node") {
    var tree = exampleTree
    print("Tree before removal:")
    print(tree)
    tree.remove(3)
    print("Tree after removing root:")
    print(tree)
}

/*
---Example of removing a node---
Tree before removal:
 ┌──5
┌──4
│ └──nil 
3
│ ┌──2
└──1
 └──0

Tree after removing root:
┌──5
4
│ ┌──2
└──1
 └──0
*/
```

## 关键点总结

* 二叉搜索树是一种存储排序数据的数据结构；
* 二叉搜索树的插入、移除和查找算法的平均时间复杂度为O(log n)；
* 当树不平衡的时候，时间复杂度会降低到O(n)。