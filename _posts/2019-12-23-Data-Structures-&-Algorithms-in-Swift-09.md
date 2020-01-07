---
layout: post
author: Robin
title: \#9\ 一般树与树节点遍历
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/9/cover.png'
---


在计算机编程的世界中，**树**是一种非常重要的数据结构。树用于解决很多计算机编程世界的挑战，例如：

* 等级关系的描述
* 分类数据的管理
* 分类查找操作

在计算机算法中，树有很多种，每一种都有其特有的形状和大小。在本文中将学习关于树的基础知识，以及使用Swfit编程语言实现树结构等。

## 术语

关于树的术语有很多，只有将各个术语的含义弄清楚之后，才能够实现树，并利用树来解决问题。

### 节点

类似链表，树也是由节点构成的。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/node.png)

每一个节点会封装一些数据，并链接着其*孩子*。

### 父节点和子节点

树的结构是从顶部延伸到底部的，看起来像一颗反过来的真实的树。

在树的结构中，除了最上方的节点之外，每一个节点都链接着它上面的节点，这个节点称之为**父节点**。除了最下方的节点之外，每一个节点都连接着它下面的节点，这个节点称之为**子节点**。在树中，每一个子节点只有一个父节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/tree.png)

### 根节点

树结构中，最顶端的节点称为**根节点**。根节点再无父节点，并且一颗树中有且仅有一个根节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/root.png)

### 叶子节点

没有子节点的节点，称之为**叶子节点**。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/leaf.png)

## Swift树结构实现

```swift
public class TreeNode<T> {
    public var value: T
    public var children: [TreeNode] = []
    
    public init(_ value: T) {
        self.value = value
    }
}
```

对于一棵树来说，最为重要的便是树的节点，每一个节点都有两个主要功能，封装数据和链接其他节点。在上述实现中，创建类TreeNode来对节点的结构进行封装，并且在节点的结构中，其所有的子节点使用了数组进行封装，数组中依然是节点结构。

对于一棵树来说，树中的节点可以进行添加，即为某个节点添加新的节点，因此添加如下方法：

```swift
// 为节点添加新的子节点
public func add(_ child: TreeNode) {
    children.append(child)
}
```

Time to give it a whirl.

```swift
example(of: "Create a tree") {
    let beverages = TreeNode("Beverages")
    let hot = TreeNode("Hot")
    let cold = TreeNode("Cold")
    
    beverages.add(hot)
    beverages.add(cold)
    
    print(beverages.value)
    print(beverages.children[0].value)
    print(beverages.children[1].value)
}

/*
---Example of Create a tree---
Beverages
Hot
Cold
*/
```

树的结构属于层级结构，上述Demo中为根节点Beverages增加了两个子节点Hot和Cold。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/create-tree.png)

## 遍历算法

线性集合（如数组、链表）的遍历相对简单，因为他们都有清晰的起点和终点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/linear-collection.png)

然而遍历一颗树相对较为复杂一点，对于一颗树来说，其起点和终点并不明晰。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/tree2.png)

由于在树种，是优先遍历左边的节点还是右边的节点，并不明确，只因面对的问题不同而策略不同。对于不同的树有着不同的遍历策略。

### 深度优先遍历

这是一种从根节点开始，直到回溯之前尽可能的遍历到树的叶子节点。

```swift
extension TreeNode {
    public func forEachDepthFirst(visit: (TreeNode) -> Void) {
        visit(self)
        children.forEach {
            $0.forEachDepthFirst(visit: visit)
        }
    }
}
```

这里使用的是递归的方式进行节点的遍历，如果不想使用递归，可以将children变量设置为栈类型。为了测试，首先我们构建一颗比较大的树：

```swift
func makeBeverageTree() -> TreeNode<String> {
    let tree = TreeNode("Beverages")
    let hot = TreeNode("hot")
    let cold = TreeNode("cold")
    
    let tea = TreeNode("tea")
    let coffee = TreeNode("coffee")
    let chocolate = TreeNode("cocoa")
    
    let blackTea = TreeNode("black")
    let greenTea = TreeNode("green")
    let chaiTea = TreeNode("chai")
    
    let soda = TreeNode("sida")
    let milk = TreeNode("milk")
    
    let gingerAle = TreeNode("ginger ale")
    let bitterLemon = TreeNode("bitter lemon")
    
    tree.add(hot)
    tree.add(cold)
    
    hot.add(tea)
    hot.add(coffee)
    hot.add(chocolate)
    
    cold.add(soda)
    cold.add(milk)
    
    tea.add(blackTea)
    tea.add(greenTea)
    tea.add(chaiTea)
    
    soda.add(gingerAle)
    soda.add(bitterLemon)
    
    return tree
}
```

该树的形态如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/demo-large-tree.png)

接下来在这棵树上测试深度优先遍历。

```swift
example(of: "depth-first traversal") {
    let tree = makeBeverageTree()
    tree.forEachDepthFirst {
        print($0.value)
    }
}

/*
---Example of depth-first traversal---
Beverages
hot
tea
black
green
chai
coffee
cocoa
cold
sida
ginger ale
bitter lemon
milk
*/
```

从上述测试打印的结果和树的形态图可以看出，深度优先遵循从左至右的原则。

### 广度优先遍历

广度优先遍历又称为水平顺序遍历，其算法如下：

```swift
extension TreeNode {
    public func forEachLevelOrder(visit: (TreeNode) -> Void) {
        visit(self)
        var queue = Array<TreeNode>()
        children.forEach {
            queue.append($0)
        }
        
        while let node = queue.isEmpty ? nil : queue.removeFirst() {
            visit(node)
            node.children.forEach {
                queue.append($0)
            }
        }
    }
}
```

这里的实现采用了数组作为临时变量，存储元素，也可以直接使用队列。

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/level-order.png)

```swift
example(of: "level-order traversal") {
    let tree = makeBeverageTree()
    tree.forEachLevelOrder {
        print($0.value)
    }
}

/*
---Example of level-order traversal---
Beverages
hot
cold
tea
coffee
cocoa
sida
milk
black
green
chai
ginger ale
bitter lemon
*/
```

### 节点搜索

上面实现了树的两种遍历算法 --- 深度优先和广度优先，分别针对了不同的特定问题。有了遍历的算法之后，针对节点的搜索而言，便无需太过复杂的算法了。

```swift
extension TreeNode where T: Equatable {
    public func search(_ value: T) -> TreeNode? {
        var result: TreeNode?
        forEachLevelOrder { node in
            if node.value == value {
                result = node
            }
        }
        return result
    }
}
```

在这个搜索算法中，使用了广度优先的遍历算法，也可使用深度优先的遍历算法。但是如果在树种有多个相匹配的节点，搜索算法最终保存的是最后一个节点。

```swift
example(of: "searching for a node") {
    let tree = makeBeverageTree()
    
    if let searchResult1 = tree.search("ginger ale") {
        print("Found node: \(searchResult1.value)")
    }
    
    if let searchResult2 = tree.search("WKD Blue") {
        print(searchResult2.value)
    } else {
      print("Couldn't find WKD Blue")
    }
}

/*
---Example of searching for a node---
Found node: ginger ale
Couldn't find WKD Blue
*/
```

## 关键点总结

* 树结构和链表类似，但是链表的每一个节点只能链接到另一个节点，而树的一个节点可以链接多个节点；
* 针对树来说，有一些特定的术语，如根节点、子节点、叶子节点等；
* 节点的遍历 --- 深度优先和广度优先并只是应用在一般的树中，其他树的结构也可使用，只不过会根据树的不同而策略不同。

## Challenge

打印树中同一层级的元素，每个相同层级的元素打印在一行中。例如：

![](/assets/Data-Structures-&-Algorithms-in-Swift/9/challenge.png)

打印的结果应该是：

```
15 
1 17 20 
1 5 0 2 5 7
```

```swift
func printEachLevel<T>(for tree: TreeNode<T>) {
    var queue = Array<TreeNode<T>>()
    var nodesLeftInCurrentLevel = 0
    queue.append(tree)
    
    while !queue.isEmpty {
        nodesLeftInCurrentLevel = queue.count
        while nodesLeftInCurrentLevel > 0 {
            guard let node = queue.isEmpty ? nil : queue.removeFirst()  else { break }
            print("\(node.value)", terminator: " ")
            node.children.forEach { queue.append($0) }
            nodesLeftInCurrentLevel -= 1
        }
        print()
    }
}
```