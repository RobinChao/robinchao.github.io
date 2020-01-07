---
layout: post
author: Robin
title: \#13\ 字典树（Tries Tree）
tags: Swift中的数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/13/cover.jpg'
---

**Tries** 是一颗用于存储可以表示为集合的数据的树，又称前缀树或字典树，是一种有序树，用于保存关联数组，其中的键通常是字符串。与二叉查找树不同，键不是直接保存在节点中，而是由节点在树中的位置决定。一个节点的所有子孙都有相同的前缀，也就是这个节点对应的字符串，而根节点对应空字符串。一般情况下，不是所有的节点都有对应的值，只有叶子节点和部分内部节点所对应的键才有相关的值。

例如利用Tries表示一个英语单词，可以表示如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/13/tries-word-eg.png)

字符串中的每一个字符被表示为一个节点，字符串中最后的节点会使用带有点号标识来标记为终止节点。通过在前缀匹配的上下文中查看字典树，会发现字典树的众多优点。

## Example

假设有一个字符串的集合，该如何构建每一个字符串的前缀匹配逻辑呢？

```swift
class EnglishDictionary {
    private var words: [String]
    
    func words(matching prefix: String) -> [String] {
        return words.filter { $0.hasPrefix(prefix) }
    }
}
```

*words(matching:)*方法将会遍历字符串集合并返回与预设前缀匹配的字符串。

当*words*数组中的字符串个数比较少的时候，上述方法是可行且高效的，但是当字符串集合中的字符串数量到达几千，上述方法仅仅在数组的遍历上就会形成性能瓶颈。上述方法的时间复杂度为O(k * n)，其中k为字符串集合中最长的字符串，n 为字符串集合中需要检查的字符串数量。

对于此类问题，Tries数据结构有着出色的性能表现，作为具有支持多个子节点的节点的树，每个节点可以代表一个字符。通过跟踪从根节点到用点号标识的特殊终止节点的集合，形成一系列的单词组合。Tries的特点也是多个预表示的结果会共享节点集合。

为了进一步的了解和说明Tries的性能，假设已有如下的Tries结构，从中找出前缀CU代表的单词。

![](/assets/Data-Structures-&-Algorithms-in-Swift/13/eg-cu-1.png)

首先，从根节点出发，找到包含字符C的节点，找到后，就可以排除一些其他的子树，例如上图中根节点的两个子树。

然后，需要以C节点开始，在其子节点中寻找包含字符U的节点，如下：

![](/assets/Data-Structures-&-Algorithms-in-Swift/13/eg-cu-2.png)

既然匹配的是前缀，因此在上图中以CU为前缀的节点将会被返回，上例中将返回CUT或CUTE。想象如果有上百上千的字符串，需要匹配前缀CU，Tries的数据结构可以避免多次的数据比较，提高匹配性能等。

![](/assets/Data-Structures-&-Algorithms-in-Swift/13/eg-cu-3.png)

## 结构实现

Tries本质上也是树型数据结构，因此会有节点，首先实现其节点的数据结构。

### TrieNode

```swift
public class TrieNode<Key: Hashable> {
    public var key: Key?
    public weak var parent: TrieNode?
    public var children: [Key: TrieNode] = [:]
    public var isTerminating = false
    
    public init(key: Key?, parent: TrieNode?) {
        self.key = key
        self.parent = parent
    }
}
```

Tries的节点结构和其他树型数据结构有明显的不同。

* **key：** 存储节点的数据。由于根节点不存储数据，因此该属性为optional类型；
* **parent：**当前节点父节点的弱引用，在节点的删除中将会利用此属性高效完成节点删除操作；
* **children：**在BST中，一个节点拥有左节点和右节点，在Tries中，一个节点会持有多个不同的元素，因此**children**被定义为字典类型；
* **isTerminating：**标记当前节点是否是集合的终止节点。

### Trie

```swift
public class Trie<CollectionType: Collection> where CollectionType.Element: Hashable{
    
    public typealias Node = TrieNode<CollectionType.Element>
    private let root = Node(key: nil, parent: nil)
    
    public init() {}
}
```
 *Trie*类是为所有采用Collection协议的类型构建的，包括*String*。除此之外，集合中的每一个元素都是可哈希的，因为集合中的每一个元素都会作为*TrieNode*中*children*的*key*。

 基本的结构完成了，接下来就是为Trie实现基本的节点操作方法，包括*insert*、*contains*、*remove*以及前缀匹配算法。

## 操作算法实现

### Insert

Trie结构可以适用于任何Collection的类型，Trie采用集合并将集合中的每一个元素表示为一个节点，节点和元素之间形成映射的关系。

```swift
public func insert(_ collection: CollectionType) {
    var current = root
        
    for element in collection {
        if current.children[element] == nil {
            current.children[element] = Node(key: element, parent: current)
        }
        current = current.children[element]!
    }
        
    current.isTerminating = true
}
```

* **current** 变量保持着对遍历进度的追踪，开始于Trie树的根节点;
* Trie树的每一个节点与集合中的每一个元素相对应。对于集合中的每一个元素，首先要检查子节点字典中是否存在当前元素，如果不存在，则创建一个新节点，之后将循环移至下一个分支节点；
* **for**循环迭代完成之后，**current**指向集合中最后一个元素，也就是current节点已经是终止节点了，此时设置其终止标志**isTerminating**为**true**。

该操作的时间复杂度为O(k)，其中 k 是待插入元素的集合中元素的个数。因为在插入算法中，需要遍历集合中的每一个元素，并可能为每一个元素创建新的节点。

### Contains

**contains** 非常类似于 **insert** 算法，其目标是检查集合中的元素在Trie中是否存在。

```swift
extension Trie {
    public func contains(_ collection: CollectionType) -> Bool {
        var current = root
        
        for element in collection {
            guard let child = current.children[element] else {
                return false
            }
            
            current = child
        }
        return current.isTerminating
    }
}
```

对集合的遍历类似于insert，如果集合中的元素在Trie中不存在，则直接返回，否则依次移动current至子节点，继续遍历检查，直到元素遍历完成，此时current节点是否为终止节点，即为返回结果。如果最终所有的元素都没有在Trie树中找到，则该集合并没有添加到Trie树中，可能其只是更大集合的一个子集而已。

该操作的时间复杂度为O(k)，同样的 k 是待查找的集合中元素的个数。因为需要对集合中的每一个元素进行遍历，以检查其是否处于Trie树中。

```swift
example(of: "insert and contains") {
    let trie = Trie<String>()
    trie.insert("cute")
    trie.insert("cut")
    if trie.contains("cute") {
        print("cute is in the trie")
    }
}

/*
---Example of insert and contains---
cute is in the trie
*/
```


### Remove

移除Trie树中的一个节点相对复杂一点，尤其当一个节点被两个不同的集合所共享的时候，需要更加的小心。

```swift
extension Trie {
    public func remove(_ collection: CollectionType) {
        var current = root
        for element in collection {
            guard let child = current.children[element] else {
                return
            }
            current = child
        }
        
        guard current.isTerminating else {
            return
        }
        current.isTerminating = false
        while let parent = current.parent, current.children.isEmpty && !current.isTerminating {
            parent.children[current.key!] = nil
            current = parent
        }
    }
}
```

* 准备移除之前的检查工作，类似于contains操作。在这里是为了检查集合是否存在于Trie树中，以及将current指向集合的最后一个节点；
* 设置current节点的*isTerminating*为false，目的是为了在下一次的循环中，节点能够被移除掉；
* 最后的while循环是相对棘手的部分。因为节点是可以被共享的，因此不希望在删除节点时误删掉另一个集合中的节点，如果当前节点再无子节点，则说明其他集合不依赖当前节点。同时还需检查当前节点是否为终止节点，如果是终止节点，则说明当前节点属于另一个集合，不能进行删除，如果不是终止节点，就可以不断的使用回溯父节点属性，并进行对应元素的删除。

该操作的时间复杂度为O(k)，其中 k  是待删除集合中元素的个数。

```swift
example(of: "remove") {
    let trie = Trie<String>()
    trie.insert("cut")
    trie.insert("cute")
    
    print("\n*** Before removeing ***")
    assert(trie.contains("cut"))
    print("\"cut\" is in the trie")
    assert(trie.contains("cute"))
    print("\"cute\" is in the trie")
    
    print("\n*** After removing cut ***")
    trie.remove("cut")
    assert(!trie.contains("cut"))
    assert(trie.contains("cute"))
    print("\"cute\" is still in the trie")
}

/*
---Example of remove---

*** Before removeing ***
"cut" is in the trie
"cute" is in the trie

*** After removing cut ***
"cute" is still in the trie
*/
```

### Prefix matching

Trie树最具标志性的算法是**前缀匹配**算法。

```swift
extension Trie where CollectionType: RangeReplaceableCollection {
    
}
```

首先对**CollectionType**进行**RangeReplaceableCollection**限制，因为在实际的操作中，需要使用**RangeReplaceableCollection**中的**append**方法。

```swift
public func collections(startingWith prefix: CollectionType) -> [CollectionType] {
    var current = root
    for element in prefix {
        guard let child = current.children[element] else {
            return []
        }
        current = child
    }
    
    return collections(startingWith: prefix, after: current)
}
```

* 首先检查Trie树中是否包含预检索的前缀，如果不包含则返回空数组；
* 当检查得到预检索的前缀后，将其所在的节点传递给辅助方法*collections(startingWith:after:)*，递归查找所有顺序。

```swift
private func collections(startingWith prefix: CollectionType, after node: Node) -> [CollectionType] {
    
    var results: [CollectionType] = []
    
    if node.isTerminating {
        results.append(prefix)
    }
    
    for child in node.children.values {
        var prefix = prefix
        prefix.append(child.key!)
        results.append(contentsOf: collections(startingWith: prefix, after: child))
    }
    return results
}
```

* 首先构建一个空的数组变量，以保存输出结果。如果当前节点是终止节点，则直接添加当前节点到结果数组中，因为预检索前缀所在的节点此时也是一个结果；
* 接下来，需要检查当前节点的子节点，针对每一个子节点，递归调用*collections(startingWith:after:)*方法，寻找其他终止节点。

*collections(startingWith:)*方法的时间复杂度为O(k * m)，其中 k 表示与前缀匹配最长的集合，m 表示与前缀匹配的集合数。数组的时间复杂度为O（k *n），其中n是集合中元素的数量。

对于每个集合中均匀分布的大量数据，与使用数组进行前缀匹配相比，Trie的性能要好得多。

```swift
example(of: "prefix matching") {
    let trie = Trie<String>()
    trie.insert("car")
    trie.insert("card")
    trie.insert("care")
    trie.insert("cared")
    trie.insert("cars")
    trie.insert("carbs")
    trie.insert("carapace")
    trie.insert("cargo")
    
    print("\nCollections starting with \"car\"")
    let prefixedWithCar = trie.collections(startingWith: "car")
    print(prefixedWithCar)
    
    print("\nCollections starting with \"care\"")
    let prefixedWithCare = trie.collections(startingWith: "care")
    print(prefixedWithCare)
}

/*
---Example of prefix matching---

Collections starting with "car"
["car", "cars", "card", "carbs", "cargo", "care", "cared", "carapace"]

Collections starting with "care"
["care", "cared"]
*/
```

## 关键点总结

* Trie树在前缀匹配上有着卓越的性能表现；
* Tries具有相对较高的内存效率，因为各个节点可以在许多不同的值之间共享。例如，“car”，“carbs”和“care”可以共享单词的前三个字母。