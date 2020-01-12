---
layout: post
author: Robin
title: \#15\ 堆数据结构（The Heap Data Structure）
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/15/cover.jpg'
---

想必抓娃娃机如今没有人不知道其实什么了，抓娃娃机的爪子总是那么的难以控制，总是看起来容易的机会却难以如愿。抓抓机的爪子其实就工作在一个堆数据结构之上，爪子每次抓的几乎都是那边一堆玩具最上面的那一个，只有这样机会才会更大一些。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/zhuawawa.png)

在本文中将学习关于堆（Heap）的基础知识，包含如何创建一个堆数据结构，如果从堆数据结构中获取最大和最小元素等。

## 什么是堆？

堆是一个使用数组构建的完整二叉树，也称为二叉堆。

> 这里的堆和内存堆是完全不同的一个概念，需要区分。在计算机科学中，经常有一些术语被重复使用，但是涵义却有所不同，本文不会对内存堆进行阐述。

堆有两种类型：

1. **最大堆：**堆中元素越大，其优先级越高；
2. **最小堆：**堆中元素越小，其优先级越高。

## 堆属性

一个堆结构，有着必须始终满足的重要特征，称之为**堆不变式**或**堆属性**。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/max-min-heap.png)

在最大堆中，父节点必须包含一个大于等于其子节点的值，根节点包含最大的值。

在最小堆中，父节点必须包含一个小于等于其子节点的值，根节点包含最小的值。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/level-heap.png)

另一个堆的必须属性是**堆是一个完全二叉树**。意味着树除了叶子节点层之外，其他每一层都必须被填充，有点类似某些闯关类游戏，本关没有完成，则下一关无法开始。

## 堆的应用

堆在很多场景下都被广泛的应用，例如：

* 计算集合中最小元素和最大元素；
* 堆排序
* 优先级队列构造
* 构造图算法，例如普林演算法 (Prim's algorithm)或狄克斯特拉算法（Dijkstra’s algorithm）等。

## 常用的堆操作

首先定义Heap的数据结构：

```swift
struct Heap<Element: Equatable> {
    var elements: [Element] = []
    let sort: (Element, Element) -> Bool
    
    init(sort: @escaping (Element, Element) -> Bool) {
        self.sort = sort
    }
}
```

在Heap的数据结构中，包含一个数组*elemtns*用来保存堆元素，一个*sort*函数定义堆中集合如何排序的排序函数。构造器接收一个适当的参数，后续用来构建最大和最小堆。

## 如何表示堆？

树型结构中的节点能够保存值和其子节点的索引，二叉树同时保存左子树和右子树的引用。堆本质上是一颗二叉树，但是可以使用简单的数组进行表示。利用数组表示堆的好处是良好的时间复杂度和空间复杂度，因为这样堆中的元素保存在内存里，堆元素的交换等能够有良好的的性能表现，与使用二叉树来表示堆，使用数组更加的容易。接下来了解使用数组如何表示一个堆。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/array-heap-tree.png)

为了使用数组表示堆，只需要从左至右一层一层迭代元素即可。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/traversal-eg.png)

当遍历进入高层级的时候，所需要遍历节点数可能会成倍的增加。

现在可以轻松访问堆中的任何节点。您可以将这一点与访问数组中元素的方式进行比较：无需向下遍历左分支或右分支，只需使用简单公式访问数组中的节点即可。

例如给定一个以零为开始索引的 *i* 对应的节点：

* 当前节点的左子树能够使用 *2i + 1* 进行访问；
* 当前节点的右子树能够使用 *2i + 2* 进行访问；

如下图：

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/index-math.png)

如果需要访问节点的父节点，依然可以使用索引值 *i* 求解，例如在索引为 *i* 的节点上，其父节点索引可通过 *floor( (i - 1) / 2)*求得。

> 在二叉树中，左子树和右子树的节点搜索需要O(log n)时间复杂度，但是通过数组的方式获取的时候，时间复杂度仅为O(1)。

了解了堆的知识后，即可继续完善堆的数据结构，并为其添加一些方便的方法：

```swift
var isEmpty: Bool {
    return elements.isEmpty
}

var count: Int {
    return elements.count
}

func peek() -> Element? {
    return elements.first
}

func leftChildIndex(ofParentAt index: Int) -> Int {
    return (2 * index) + 1
}

func rightChildIndex(ofParentAt index: Int) -> Int {
    return (2 * index) + 2
}

func parentIndex(ofChildAt index: Int) -> Int {
    return (index - 1) / 2
}
```

## 从堆中移除元素

最基本的元素节点移除操作是移除根节点，例如下图所示的移除最大堆中的根节点10：

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/remove-max-heap.png)

此时，移除操作将移除位于根节点的集合最大值。首先要使用堆中最末尾的元素和根节点进行交换，一旦交换了元素，就可以删除位于叶子节点上的需要删除的元素了。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/delete-leaf-node.png)

但是，删除后的堆还是最大堆结构么？需要注意的是，最大堆的原则或者规则是每一个子节点的值都小于或等于父节点的值，一旦不符合这个规则，则需要进行节点的**sift down**调整。（最大堆调整算法称为**sift down**，最小堆调整算法称为** sift up**）

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/sift-down.png)

针对上图所示，**sift down**调整的方法是，获取根节点元素3，判断和其左子节点和右子节点的大小，如果左子节点的值大于当前节点，则进行节点的交换，如果左子节点和右子节点均大于该值，则使用子节点中大的那个值和当前节点进行交换。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/sift-down-2.png)

继续使用**sift down**调整法，调整节点，直到所有的节点满足最大堆的规则。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/sift-down-done.png)

### 算法实现

```swift
mutating func remove() -> Element? {
    guard !isEmpty else {
        return nil
    }
    
    elements.swapAt(0, count - 1)
    
    defer {
        siftSown(from: 0)
    }
    return elements.removeLast()
}
```

* 首先检查堆是否为空，如果为空，则返回*nil*；
* 交换根节点和堆中最后的元素位置；
* 移除集合中最后一个元素并返回该元素（最后一个元素不是最大值就是最小值）；
* 移除后，堆可能不符合最大堆或最小堆的原则，需要继续采用siftDown或者siftUp方法进行调整，直到符合堆的规则。

```swift
mutating func siftSown(from index: Int) {
    var parent = index
    while true {
        let left = leftChildIndex(ofParentAt: parent)
        let right = rightChildIndex(ofParentAt: parent)
        var candidate = parent
        
        if left < count && sort(elements[left], elements[candidate]) {
            candidate = left
        }
        if right < count && sort(elements[right], elements[candidate]) {
            candidate = right
        }
        if candidate == parent {
            return
        }
        elements.swapAt(parent, candidate)
        parent = candidate
    }
}
```

**siftDown(from:)**接受任意的索引，并将其视为根节点，该方法的工作原理是：

1. 临时保存索引到变量*parent*；
2. 一直进行sifting操作，直到return（while true）；
3. 获取*parent*索引所在节点的左节点和右节点对应的索引；
4. 使用临时变量*candidate*追踪和父节点进行交换的节点索引；
5. 如果是左节点，并且左节点相比父节点有更高的优先级，则*candidate*为左节点；
6. 如果是右节点，并且右节点相比父节点有更高的优先级，则*candidate*为右节点；
7. 如果*candidate*依然是*parent*，说明已经调整到末尾，再无sifting的必要了；
8. 一轮sifting结束时，重新设定parent为候选的*candidate*，进行下一轮的sifting。

## 向堆中插入元素

假设需要向如下的堆中插入元素7：

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/insert-origin.png)

首先将待插入的元素添加到堆的末端：

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/insert-end.png)

之后，检查最大堆的堆属性。和*siftdown*不同的是，此时使用*siftup*方法，工作原理类似于*siftdown*，通过比较当前节点和其父节点进行节点的交换。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/siftup.png)

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/siftup-done.png)

### 算法实现

```swift
extension Heap {
    mutating func insert(_ element: Element) {
        elements.append(element)
        siftUp(from: elements.count - 1)
    }
    
    mutating func siftUp(from index: Int) {
        var child = index
        var parent = parentIndex(ofChildAt: child)
        while child > 0 && sort(elements[child], elements[parent]) {
            elements.swapAt(child, parent)
            child = parent
            parent = parentIndex(ofChildAt: child)
        }
    }
}
```

插入算法相较于移除算法，更为直接：

* 首先直接向数组中追加待插入的元素，之后进行 **sift up** 调整；
* *siftUp*比较当前节点和其父节点，并进行条件进行交换，直到该节点有一个比其父节点更高的优先级为止。

在从堆中移除元素的时候，删除算法只是移除了堆的根节点，但是非根节点的元素移除可能更加的符合实际的场景。

## 从任意索引中删除

```swift
extension Heap {
    mutating func remove(at index: Int) -> Element? {
        guard index < elements.count else {
            return nil
        }
        
        if index == elements.count - 1 {
            return elements.removeLast()
        } else {
            elements.swapAt(index, elements.count - 1)
            defer {
                siftSown(from: index)
                siftUp(from: index)
            }
            return elements.removeLast()
        }
    }
}
```

* 检查待删除的索引是否在集合的边界之内，如果不在，返回*nil*；
* 如果删除的是堆中最末尾的元素，则直接进行删除，类似*remove*；
* 如果是非末尾的元素，首先交换待删除索引和末尾索引；
	* 之后删除末尾的元素，并返回该元素
	* 最后，调用*siftDown*和*siftUp*进行堆节点调整

但是为什么要同时调用*siftDown*和*siftUp*呢？

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/sift-both.png)

例如上图所示的堆中，想要删除元素5，首先交换5和最末尾的元素8，之后删除元素5。此时需要使用*sift up*对最大堆属性进行调整。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/sift-swap.png)

例如上图，想要删除元素7，需要和末尾元素1进行交换后删除，删除后，需要使用*sift down*进行调整。

## 在堆中搜索元素

在删除元素之前，首先要通过索引查找对应的元素，此时需要进行堆元素的搜索。不过，堆本身并没有设计快速的搜索，对于一颗二叉搜索树来说，搜索元素有O(log n)的时间复杂度，但是对于使用数组构建的堆，数组中的元素进行排序确实不同于二叉搜索树的，此时并不能使用二分查找。

> 在堆中搜索元素最差的情况下有O(n)的时间复杂度，因此在搜索的时候，可能要检查数组中的每一个元素。

```swift
extension Heap {
    func index(of element: Element, startingAt i: Int) -> Int? {
        if i >= count {
            return nil
        }
        if sort(element, elements[i]) {
            return nil
        }
        if element == elements[i] {
            return i
        }
        if let j = index(of: element, startingAt: leftChildIndex(ofParentAt: i)) {
            return j
        }
        if let j = index(of: element, startingAt: rightChildIndex(ofParentAt: i)) {
            return j
        }
        return nil
    }
}
```	

* 如果索引i大于或者等于堆元素个数，则搜索失败，返回*nil*；
* 判断当前元素是否比索引i所对应的元素有更高的优先级，如果是，则所搜索的元素不可能在堆的更低的索引；
* 如果所搜索的元素和索引i所对应的元素相等，则待删除元素所在的索引就是i；
* 递归的搜索左子树从索引i开始的元素；
* 递归的搜索右子树从索引i开始的元素；
* 如果上述过程全部搜索失败，则整个搜索失败。

## 构建堆

至此，已经有足够的工具针对堆进行各类操作了，但是还有一个问题就是，如果使用已存在的数组构建一个堆？在开始定义堆数据结构的时候，我们使用了一个非常简单的构造器，对其进行改造如下：

```swift
init(sort: @escaping (Element, Element) -> Bool, elements: [Element] = []) {
    self.sort = sort
    self.elements = elements
    
    if !elements.isEmpty {
        for i in stride(from: elements.count / 2 - 1, through: 0, by: -1) {
            siftSown(from: i)
        }
    }
}
```

构造器现在能够接收一个额外的参数，如果传入一个非空的数组，将会使用该数组构建堆，为了使得堆满足堆属性，从第一个非叶节点开始向后循环数组，然后筛选所有父节点。您只遍历了一半的元素，因为筛选叶节点没有点，只有父节点。

![](/assets/Data-Structures-&-Algorithms-in-Swift/15/build-heap-with-array.png)

## 测试

```swift
example(of: "building a heap with array") {
    var heap = Heap(sort: >, elements: [1, 12, 3, 4, 1, 6, 8, 7])
    
    while !heap.isEmpty  {
        print(heap.remove()!)
    }
}

/*
---Example of building a heap with array---
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

* 对于堆的各类操作，有着不同的时间复杂度，具体如下：
	![](/assets/Data-Structures-&-Algorithms-in-Swift/15/heap-time.png)
* 堆数据结构非常适合维护优先级最高或最低优先级的元素。
* 每次从堆中插入或删除项时，都必须检查它是否符合优先级的规则。
