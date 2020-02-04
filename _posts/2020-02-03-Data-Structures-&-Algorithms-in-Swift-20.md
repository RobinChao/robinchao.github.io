---
layout: post
author: Robin
title: \#20\ 堆排序（Heap Sort）
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/20/cover.jpg'
---

**堆排序[Heap Sort]**是另一种基于比较的排序算法，其利用堆对数组进行升序排序。关于堆数据结构，可以查看[\\#15\ 堆数据结构（The Heap Data Structure）](https://robinchao.github.io/Data-Structures-&-Algorithms-in-Swift-15/)中的介绍。

堆排序使用的是堆的优势，根据堆的定义，一个部分排序的二叉树具有如下的特质：

1. 在最大堆中，所有的父节点均大于其孩子节点；
2. 在最小堆中，所有的父节点均小于其孩子节点。

最大堆和最小堆的图示如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/max-min-heap.png)

## 示例

对于给定的未排序的数组，从小到大进行排序，堆排序都必须首先将该数组转换为最大堆结构。

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/eg-1.png)

对上述数组通过筛选所有父节点进行转换，此时使用sift-down方式，最终转换后的结果如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/eg-2.png)

对应的数组为：

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/eg-3.png)

由于单次sift-down操作的时间复杂度为O(log n)，因此构建一个堆的整体时间复杂度为O(n log n)。

堆排序是将数组元素进行升序排序。因为在最大堆中，最大的元素通常位于根节点，因此可以**使用索引0的元素和索引n-1的元素进行直接交换**。这样交换后，数组最后的元素便位于正确地位置，但是此时堆已经不符合堆的规则了。下一步对新的根节点元素进行sift-down操作，使得堆成立。（此时进行sift-down的时候，需要将排除部分已排序好的元素）

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/eg-4.png)

对元素5进行sift-down之后，新的根节点为原始堆中第二大的元素21，此时同样和末尾元素6进行交换，交换后继续对新的根节点6进行sift-down操作，再次使得堆成立。

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/eg-5.png)

上述过程其实形成了一种模式，堆排序简单直接，每次交换首末两个元素，较大的元素依次被交换到数组的后面，多次交换完成后，数组变成了从小到大的顺序，也完成了堆排序。

![](/assets/Data-Structures-&-Algorithms-in-Swift/20/eg-6.png)

## 算法实现

堆排序的实现是基于堆的数据结构基础上的，是对堆结构的一种功能扩展。

```swift
extension Heap {
    
    mutating func siftSown(from index: Int, upTo size: Int) {
           var parent = index
           while true {
               let left = leftChildIndex(ofParentAt: parent)
               let right = rightChildIndex(ofParentAt: parent)
               var candidate = parent
               
               if left < size && sort(elements[left], elements[candidate]) {
                   candidate = left
               }
               if right < size && sort(elements[right], elements[candidate]) {
                   candidate = right
               }
               if candidate == parent {
                   return
               }
               elements.swapAt(parent, candidate)
               parent = candidate
           }
       }
    
    func sorted() -> [Element] {
        var heap = Heap(sort: sort, elements: elements)
        for index in heap.elements.indices.reversed() {
            heap.elements.swapAt(0, index)
            heap.siftSown(from: 0, upTo: index)
        }
        return heap.elements
    }
}
```

> 需要对原来Heap结构的sift-down方法进行改造，增加参数size以标记当前集合的大小。

堆排序算法工作流程如下：

1. 首先对原有堆进行一个拷贝。因为在堆排序堆元素集合进行排序后，原有的堆结构将不再成立，为了保持堆结构成立，这里使用其拷贝进行排序；
2. 从集合末尾元素开始，对集合进行遍历；
3. 交换首末位置的元素，此次交换后，最大的元素将位于集合的末尾；
4. 交换元素位置后，堆结构已经不成立了，因此需要使用sift-down方法对集合重新调整，已重生合法的堆结构，完成后，新的根节点将是原集合中第二大的元素。重复第三步即可。

```swift
example(of: "heap sort") {
    let heap = Heap(sort: >, elements: [6, 12, 2, 26, 8, 18, 21, 9, 5])
    print(heap.sorted())
}

/*
---Example of heap sort---
[2, 5, 6, 8, 9, 12, 18, 21, 26]
*/
```

## 性能

堆排序的最佳、最差和平均性能都是O(n log n)。因为必须遍历整个列表一次，并且每次交换元素时，都必须执行向下筛选sift-down操作，这是一个O(log n)操作。

堆排序也不是一种稳定的排序，因为它取决于元素如何布局和放入堆中。例如，如果您正在根据一副纸牌的等级对其进行堆排序，您可能会看到它们的套件相对于原始纸牌的顺序发生了变化。

## 关键点总结

* 堆排序利用最大堆数据结构对数组中的元素进行排序。