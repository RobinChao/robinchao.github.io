---
layout: post
author: Robin
title: \#8\ 队列的Swift实现与操作定义
tags: 数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/8/cover.jpg'
---

在生活中，人人都熟悉排队等待。无论你是在排队购买喜欢的电影的电影票，还是排队等待打印一份文件等等，这些都是**队列（Queue）**数据结构。在上文[\\#7\ Stack & Stack Simple Challenges](https://robinchao.github.io/2019/12/08/Data-Structures-&-Algorithms-in-Swift-07.html)中已经提到过队列和栈属于基本的数据结构类型，但是其在应用层面非常有效。

**队列（Queue）**是一种**FIFO(ﬁrst-in ﬁrst-out)**型的数据操作特性，和栈的**LIFO**形成鲜明的对比。**FIFO**意味着首先进入队列的元素，也是第一个推出队列的元素。在项目中，如果要维护一个有顺序的数据并稍后处理，队列是无二之选。

在本内容中，我们将学习关于队列的常见操作，以及使用Swift语言实现这些操作，衡量这些操作的时间复杂度等。

## 一般性操作实现

由于队列的操作特性较多，在这里我们可以使用Swift的面向协议的编程思想进行实现，首先我们定义一个关于操作的协议，如下：

```swift
public protocol Queue {
    associatedtype  Element
    mutating func enqueue(_ element: Element) -> Bool
    mutating func dequeue() -> Element?
    var isEmpty: Bool { get }
    var peek: Element? { get }
}
```

* **enqueue：** 向队列的尾部插入一个元素，如果该操作成功，则返回true，反之返回false；
* **dequeue：** 从队列的头部删除一个元素，并返回被删除的元素；
* **isEmpty：** 检查队列是否为空；
* **peek：** 返回队列头部的元素，和**dequeue**的区别在于，该操作并不删除元素。

通过操作类型的定义可以看到，队列有两个普遍的操作，在队列的尾部插入元素和从队列的头部删除元素，而并不需要关心队列的中间元素，如果需要关心中间元素，你可能需要使用数组。

## 一个队列的例子

理解队列最简单的方式是通过实际的示例了解队列的工作原理。假设在影院门口，很多人在排队购买电影票：

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/queue-example.png)

队伍中有Ray、Brian、Sam和Mic四人，当Ray买到了电影票后，他就会从队伍中退出，相当于调用了**dequeue()**，从队伍的头部删除了一个元素类似。

此时，调用**peek**将返回队列中此刻的头元素Brain。

如果来了一个新的人Vicki，加入到了队伍中，等待购买电影票，她站到了队伍的尾部。相当于调用了**enqueue("Vicki")**。

这就是队列的一般性工作原理，接下来我们将使用四种不同的基础数据结构来创建队列以及队列的一般性操作。分别为：

* 使用数组（Array）
* 使用双向链表（Double LinkedList）
* 使用环形缓冲器（Ring buffer）
* 使用两个栈（two stacks）

##  基于Array的队列

Swift标准库中继承了大量高度优化的核心数据结构，利用这些数据结构可以构建更高级别的抽象，例如基础数据结构Array，用于存储连续的有序元素列表。在本节中，将使用Array来构建队列，并实现队列的基础操作等。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/array-queue.png)

```swift
public struct QueueArray<T>: Queue {
    private var array: [T] = []
    public init() {}
}
```

这里定义了一个采用Queue协议的通用型QueueArray结构体，在Queue协议中定义的关联类型Element这里由T推断。

接下来实现Queue协议中的定义等，使得QueueArray符合Queue协议。

### 数组检查

首先添加如下两个协议属性的实现：

```swift
public var isEmpty: Bool {
    return array.isEmpty
}
    
public var peek: T? {
    return array.first
}
```

因为这里使用的是数组，因此在实现`isEmpty`和`peek`时，均可直接使用数组的内置属性，简洁方便。其中`peek`返回的是队列的头部元素，也就是数组的第一个元素。

这连个操作的时间复杂度均为**O(1)**。

### 入队

入队就是将元素添加到队列的末尾。使用数组实现也非常方便，只要进行append操作即可。

```swift
public mutating func enqueue(_ element: T) -> Bool {
    array.append(element)
    return true
}
```

入队的操作，无论数组的大小如何，该操作的时间复杂度都是**O(1)**。这是因为在数组的末尾，存在着空白空间。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/array-queue-enqueue.png)

在上例中，当添加了`Mic`元素之后，数组还剩下两个空白空间。当添加了多个元素之后，数组的空白空间将会被填满，继续添加元素的时候，则要使用超出数组原始分配空间的空间，进而必须调整数组大小以增加空间。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/array-queue-enqueue-full.png)

在数组进行大小重新调整的时候，其时间复杂度为O(n)，数组大小重组意味着数组需要重新分配内存空间，并将原数据元素拷贝到新的数组中，因为这样的调整并不是经常性的，因此入队操作的时间复杂度仍可认为是**O(1)**。

### 出队

出队操作是将队列头节点的元素移出队列，可使用数组的**removeFirst**操作即可。

```swift
public mutating func dequeue() -> T? {
    return isEmpty ? nil : array.removeFirst()
}
```

如果数组为空，则出队操作后返回nil，否则返回出队的元素。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/array-queue-dequeue.png)

从队列中头部移除元素的操作属于O(n)时间复杂度，在上述方式中就是从数组中移除第一个元素。这始终是一个线性的时间度量，因为在内存中，当移除一个元素后，其他所有的元素都需要移动其位置。

### 调试与测试

对于调试目的来说，Swift中提供了专用的协议**CustomStringConvertible**，为了调试的方便，我们需要添加如下的代码：

```swift
extension QueueArray: CustomDebugStringConvertible {
    public var description: String {
        return String(describing: array)
    }
}
```

接下来进行队列的调试，调试代码如下：

```swift
example(of: "Debug the Queue with Array") {
    var queue = QueueArray<String>()
    queue.enqueue("Ray")
    queue.enqueue("Brian")
    queue.enqueue("Eric")
    print(queue)
    queue.dequeue()
    print(queue)
    queue.dequeue()
    print(queue.peek ?? "")
}

/*
---Example of Debug the Queue with Array---
["Ray", "Brian", "Eric"]
["Brian", "Eric"]
Eric
*/
```

由于是使用Array来进行队列的设计，因此操作方式非常类似于Array，上述打印结果也符合队列的先进先出的原则。

### 优势和劣势

上述就是基于数组的队列的一般操作的算法实现，大多数的操作都是恒定时间复杂度的，例如*dequeue()*操作，属于线性时间，内存空间也是线性的。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/array-queue-adv-disa.png)

基于数组的队列相对是简单的，也是因为数组的append操作，使得入队操作是恒定的时间复杂度O(1)。然而在实施中却有一些明显的缺点，出队的操作是从队列的头部移除元素，移除后，其他的所有元素都需要向前移动一个位置，对于队列来说影响算是非常大的。一旦队列已满，队列就必须调整其大小，调整完后，队列中很容易存在未使用的空间，随着时间的推移，未使用空间越来越多，这可能增加内存的占用率。

## 基于双向链表（Double LinkedList）的队列

在[\\#5\ Swift集合协议在Linked List上的应用](https://robinchao.github.io/2019/12/04/Data-Structures-&-Algorithms-in-Swift-05.html)中我们已经了解了单向链表，双向链表则是每个节点不仅包含指向下一个节点的指针，还包含指向上一个节点的指针。

```swift
public class LinkedListNode<T> {
    var value: T
    var next: LinkedListNode? // 指向下一个节点。尾节点为nil
    weak var previous: LinkedListNode? // 指向上一个节点。头节点为nil
        
    public init(value: T) {
        self.value = value
    }
}
```

利用双向链表实现队列如下：

```swift
public class QueueLinkedList<T>: Queue {
    private var list = DoublyLinkedList<T>()
    public init() {}
    
    public func enqueue(_ element: T) -> Bool {
        list.append(element)
        return true
    }
    
    public func dequeue() -> T? {
        guard !list.isEmpty, let _ = list.first else {
            return nil
        }
        return list.remove(at: 0)
    }
    
    public var peek: T? {
        return list.head?.value
    }
    
    public var isEmpty: Bool {
        return list.isEmpty
    }
}
```

### 入队操作

由于链表中实现了能够直接添加元素到链表尾部的操作，因此入队操作相当于链表的追加操作。

```swift
public func enqueue(_ element: T) -> Bool {
    list.append(element)
    return true
}
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-linkedlist-enqueue.png)

基于双向链表的队列入队时，其内部需要转换节点的两个指针的指向，上一个节点中指向下一个节点的指针指向该新节点，该新节点的上一个节点指向上一个节点。同时tail节点的上一个节点指针也需要更新。

### 出队操作

出队列操作前，需要检查队列是否为空队列，如果是空队列的时候，直接返回nil，否则移除链表中索引为0的元素即可。

```swift
public func dequeue() -> T? {
    guard !list.isEmpty, let _ = list.first else {
        return nil
    }
    return list.remove(at: 0)
}
```

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-linkedlist-dequeue.png)

出队需要更新head节点的指针指向，将head的next指针的指向原来队列的第二个节点即可。

### 调试与测试

同样调试模式下，我们实现**CustomStringConvertible**协议。

```swift
extension QueueLinkedList: CustomStringConvertible {
    public var description: String {
        return String(describing: list)
    }
}
```

在测试程序中实现和基于数组的队列相同的逻辑。

```swift
example(of: "Debug the Queue with Doubly Linkedlist") {
    let queue = QueueLinkedList<String>()
    queue.enqueue("Ray")
    queue.enqueue("Brian")
    queue.enqueue("Eric")
    print(queue)
    queue.dequeue()
    print(queue)
    queue.dequeue()
    print(queue.peek ?? "")
}

/*
---Example of Debug the Queue with Doubly Linkedlist---
[Ray, Brian, Eric]
[Brian, Eric]
Eric
*/
```

### 优势和劣势

基于双向链接的队列，各个操作的最佳时间复杂度和最差时间复杂度如下图所示：

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-linkedlist-queue-adv-disa.png)

基于数组的队列在出队操作上是一个线性的操作，使用链表的队列，出队操作简化为了恒定时间复杂，每次出队操作只需要更新节点的上一个和下一个指针即可。

从上表中可以看出，基于链表的队列的弱点并不明显，但是O(1)的时间复杂度却只是表面性能，操作在执行的时候，需要很高的内存开销，每个元素的操作都必须有额外的空间以供前向指针和后向指针引用，此外，每次创建新元素都需要进行昂贵的内存动态分配，相比之下，基于数组的队列进行的是批量的内存分配，速度更快。

## 基于环形缓冲器的队列

环形缓冲区也称为循环缓冲区，是一个固定大小的数组，当数组末尾没有要删除的元素时，环形缓冲区会从策略绕到数组开头。那么环形缓冲区是如何实现队列的操作的呢？

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-queue-1.png)

首先创建一个固定大小为4的缓冲区，在该缓冲区中同时含有两个指针，分别追踪不同的事情：

* **read**指针追踪队列的头部
* **write**指针追踪下一个可写的空间指针，这样就能够重写已读过的元素了。

进行入队操作，如：

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-queue-enqueue.png)

每次添加一个元素到队列的时候，**write**指针加一。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-queue-enqueue-2.png)

上图中，**write**指针又移动了两个位置，而且其位置位于**read**指针的前面，也意味着队列是非空队列。

接下来，进行两次出队的操作：

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-queue-dequeue.png)

出队操作的是**read**指针，read指针向后移动，指向第三个元素的位置即可。接下来在进行入队操作，将队列填充满：

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-queue-enqueue-full.png)

当**write**指针到达队列的末尾是，环形缓冲区会重新转换该指针到队列的开始位置。

最后，在出队两个队列中的元素：

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-queue-dequeue-2.png)

此时**read**指针也指向了队列的开始位置。最后一次出队操作后，read指针和write指针都指向了队列的开始位置，这也意味着队列中已经无元素了，为空的队列。

上述就是RingBuffer的基本数据结构和工作原理，接下来进行数据结构的实现和基本操作的实现。

首先定义所需的变量，包括了数据存储的结构，这里使用Array即可，还有两个基本的指针read和write，为了方便对数据进行检验，增加辅助检查可写空间大小和可读空间大小的变量，以及是否为空和是否已满的变量：

```swift
public struct RingBuffer<T> {
    public var array: [T?]
    public var readIndex = 0
    public var writeIndex = 0
    
    public init(count: Int) {
        array = [T?](repeating: nil, count: count)
    }
    
    public var availableSpaceForReading: Int {
      return writeIndex - readIndex
    }

    public var isEmpty: Bool {
      return availableSpaceForReading == 0
    }

    public var availableSpaceForWriting: Int {
      return array.count - availableSpaceForReading
    }

    public var isFull: Bool {
      return availableSpaceForWriting == 0
    }
}
```

由于RingBuffer是固定大小的数据结构，因此在可读可写空间判断的时候，直接使用减法的方式即可获取到可用空间大小。接下来就是基本的read操作和write操作的实现，在实现这两个操作时需要检查缓存区是否已满和是否为空：

```swift
    @discardableResult
    public mutating func write(_ element: T) -> Bool {
        guard !isFull else { return false }
        defer {
            writeIndex += 1
        }
        array[wrapped: writeIndex] = element
        return true
    }
    
    public mutating func read() -> T? {
        guard !isEmpty  else { return nil }
        defer {
            array[wrapped: readIndex] = nil
            readIndex += 1
        }
        return array[wrapped: readIndex]
    }
```

另外在操作中使用了Array的subscript操作属性，但是原始的subscript并不符合RingBuffer的定义，因此还需要重写subscript操作如下：

```swift
private extension Array {
    subscript (wrapped index: Int) -> Element {
        get {
            return self[index % count]
        }
        set {
            self[index % count] = newValue
        }
    }
}
```

另由于RingBuffer还应该支持序列的可遍历迭代操作，因此定义RingBuffer的Iterator操作：

```swift
extension RingBuffer: Sequence {
    public func makeIterator() -> AnyIterator<T> {
        var index = readIndex
        return AnyIterator {
            guard index < self.writeIndex else { return nil }
            defer {
                index += 1
            }
            return self.array[wrapped: index]
        }
    }
}
```

完成了RingBuffer的定义之后，就可以实现基于RingBuffer的队列定义和实现了。

```swift
public struct QueueRingBuffer<T>: Queue {
    
    private var ringBuffer: RingBuffer<T>
    
    public init(count: Int) {
        ringBuffer = RingBuffer<T>(count: count)
    }
    
    public var isEmpty: Bool {
        return ringBuffer.isEmpty
    }
    
    public var peek: T? {
        return ringBuffer.first as? T
    }
    
    
    public mutating func enqueue(_ element: T) -> Bool {
        return ringBuffer.write(element)
    }
    
    public mutating func dequeue() -> T? {
        return isEmpty ? nil : ringBuffer.read()
    }
}
```

队列的定义都大同小异，同样对QueueRingBuffer进行测试如下：

```swift
example(of: "Debug the Queue with RingBuffer") {
    var queue = QueueRingBuffer<String>(count: 10)
    queue.enqueue("Ray")
    queue.enqueue("Brian")
    queue.enqueue("Eric")
    print(queue)
    queue.dequeue()
    print(queue)
    queue.dequeue()
    print(queue.peek ?? "")
}

/*
---Example of Debug the Queue with RingBuffer---
RingBuffer<String>(array: [Optional("Ray"), Optional("Brian"), Optional("Eric"), nil, nil, nil, nil, nil, nil, nil], readIndex: 0, writeIndex: 3)
RingBuffer<String>(array: [nil, Optional("Brian"), Optional("Eric"), nil, nil, nil, nil, nil, nil, nil], readIndex: 1, writeIndex: 3)
*/
```

### 优缺点

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/ring-buffer-adc-disa.png)

基于环形缓冲区的队列具有相同的时间复杂性，和链表的入队和出队类似。唯一的区别是空间复杂度。环形缓冲区的大小是固定的，这意味着排队可能会失败。

到目前为止，您已经看到了三种实现：简单数组、双链表和环形缓冲区。

尽管它们看起来非常有用，但接下来您将看到使用两个栈实现的队列。您将看到它的空间位置如何远远优于链接列表。它也不需要像环形缓冲区那样的固定大小。

## 基于双栈的队列

```swift
public struct QueueStack<T>: Queue {
    private var leftStack: [T] = []
    private var rightStack: [T] = []
    public init() {}
}
```

双栈的思路其实很简单，无论何时加入元素都是讲元素添加到rightStack。当需要进行出队操作时，反转rightStack并将元素加入到leftStack中，然后在leftStack中即可使用FIFO原则进行出队操作了。

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-stack-queue.png)

根据上图双栈的工作原理，实现队列的基本操作：

```swift
    public var isEmpty: Bool {
        return leftStack.isEmpty && rightStack.isEmpty
    }
    
    public var peek: T?{
        return !leftStack.isEmpty ? leftStack.last : rightStack.first
    }
    
    public mutating func enqueue(_ element: T) -> Bool {
        rightStack.append(element)
        return true
    }
    
    public mutating func dequeue() -> T? {
        if leftStack.isEmpty {
            leftStack = rightStack.reversed()
            rightStack.removeAll()
        }
        return leftStack.popLast()
    }
```

**入队操作**

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-stack-queue-enqueue.png)


**出队操作**

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-stack-queue-dequeue.png)

测试代码如下：

```swift
example(of: "Debug the Queue with Double Stack") {
    var queue = QueueStack<String>()
    queue.enqueue("Ray")
    queue.enqueue("Brian")
    queue.enqueue("Eric")
    print(queue)
    queue.dequeue()
    print(queue)
    queue.dequeue()
    print(queue.peek ?? "")
}

/*
---Example of Debug the Queue with Double Stack---
["Ray", "Brian", "Eric"]
["Brian", "Eric"]
Eric
*/
```

### 优缺点

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/double-stack-adv-disa.png)


与基于数组的实现相比，通过利用两个堆栈，您可以将出队操作转换为分步的O(1)操作。

此外，两个栈实现是完全动态的，并且没有基于环形缓冲区的队列所具有的固定大小限制。

最后，它在空间位置方面胜过了链表。这是因为数组元素在内存块中彼此相邻。因此，在第一次访问时，大量元素将加载到缓存中。

**基于双栈的队列**

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/queue-double-stack.png)

**基于链表的队列**

![](/assets/Data-Structures-&-Algorithms-in-Swift/8/queue-linked-list.png)

## 关键点总结

* 队列是一个FIFO的结构；
* 入队操作必须在队列的末尾；
* 出队操作必须在队列的开端；
* 数组中的元素在内存中是连续的，链表是分散的，并且链表的内存存储方式可能导致缓存未命中；
* 基于环形缓冲区的队列适用于固定大小的队列结构；
* 相比其他的数据结构，双栈结构的队列能够将出栈操作分散为O(1)的时间复杂度；
* 双栈的操作在空间复杂度上由于链表结构。