---
layout: post
author: Robin
title: Swift数组安全索引扩展
tags: 开发知识 iOS
categories:
  - 开发知识
--- 


在Swift中数组和字典中下标是非常常见的，数组可以通过下标进行元素的查询和指定，字典可以通过键下标获取对应的值，也可以指定键下标对应的值。

在使用数组的时候，最致命的硬伤就是数组下标越界，如果程序中的数组越界了，导致的结果直接就是崩溃，为了防止越界引起的崩溃，可以针对集合类型的数据类型进行一个索引安全处理。比如对数组进行扩展，实现对数组索引的安全性检查，保证下标值在当前数组下标的安全范围内。

****

**一. 安全的索引集合元素**

对于一个集合索引进行安全检查是很有必要的，也非常的实用，最常见的就是对数组和字典索引进行安全检查。

1. Objective-C中对NSArray进行索引安全扩展

```
- (id)objectAtIndexSafe:(NSUInteger)index {
    if (index > self.count-1) {
        return nil;
    }
    return [self objectAtIndex:index];
}
```

2. Swift中对Array的安全扩展

上面简单的对Objective-C中的安全方法进行了简单的介绍，就算是对Swift相关内容的引子吧，下方将会给出Swift语言中类似的方法。对Swift相关方法介绍时，我会尽量的详细一些，因为毕竟本篇博客主要是关于Swift内容的。接下来将对上面Objective-C中NSArray数组索引安全验证的方法使用Swift语言进行重新。当然重写的内容也是非常容易理解的。

（1）主要是对subscript方法进行重载，在重载的subscript方法中，对index的范围通过三目运算符进行了安全检查。如果index在0..<count这个半开区间内，那么就返回当前索引的值，如果不在该范围内就返回nil, 下方就是对Array索引的安全检查。

```
extension Array {
    subscript (safe index: Int) -> Element? {
        return (0 ..< count).contains(index) ? self[index] : nil
    }
}
```

（2）上面是对Swift中的Array进行了安全索引扩展，接下来就是简单的使用了，下方的代码段是对上面安全扩展函数的测试。首先创建了一个数组testArray, 然后创建了一个索引数组indexs, 然后遍历indexs中的元素值，将其作为testArray的下标，对testArray进行检索。当然检索时，使用的是我们上面定义的safe方法，并且在indexs下标数组中存在非法的下标。在这种情况下，我们来验证一下我们的安全方法。

当然在数组遍历中，我们使用了for-in循环取出indexs中的每个index, 然后使用guard语句取出testArray中的值。使用guard语句能很好的过滤掉因为非法的index而返回的nil值。具体代码段如下所示：

```
    //MARK: - Safe Subscript Array
    func safeSubscriptArray() {
        let testArray = [0 , 2,  3, 45, 5, 23, 43, 13, 54]
        let indexs = [1, 30, 20, 2, 4, 0, 300]
        
        for index in indexs {
            guard let value = testArray[safe: index] else {
                
                print("index: \(index) is unvalid")
                continue
            }
            
            print("index: \(index)  -> value: \(value) is valid")
        }
    }
```

上面的代码段理解起来并不难，上述测试代码的运行结果如下所示，从运行结果可以很好的说明问题，并且在index非法时不会崩溃，并合理的给出相应的错误提示，请看下方具体运行结果。

```
index: 1  -> value: 2 is valid
index: 30 is unvalid
index: 20 is unvalid
index: 2  -> value: 3 is valid
index: 4  -> value: 5 is valid
index: 0  -> value: 0 is valid
index: 300 is unvalid

```

上面的延展也可以通过对整个集合类型，也就是CollectionType进行扩展，不过在扩展CollectionType时要对Index使用where子句进行限制，使Index必须符合Comparable协议，具体实现如下所示，不过下面的方法比较少用，因为一般是数组存在越界的情况，因为在字典中，如果你对一个不存在的键进行值的索引，会返回nil值，而不会崩溃。但是在数组中，你对不存在的index进行索引，就会抛出错误。下方是另一种处理方式，不过该方式用的比较少。

实现下方延展后，同样可以在数组中使用safe方法。

```
extension CollectionType where Index: Comparable {
    subscript (safe index: Index) -> Generator.Element? {
        guard startIndex <= index && index < endIndex else {
            return nil
        }
        return self[index]
    }
}
```


**二. 使用多个索引下标的数组**

延展的功能是非常强大的，该部分将会给出另一个数组的延展。该延展的功能是可以通过多个索引给数组设置值，以及通过多个索引一次性获取多个数组的值。该功能是非常强大的，接下来将一步步实现该功能。

1. 了解zip()函数以及Zip2Sequence

在实现数组多个索引扩展时，需要使用到zip()函数，zip()函数接收两个序列，并且返回一个Zip2Sequence类型的数据。zip()函数究竟是干嘛的呢？接下来将会通过一个小的实例来搞一下zip()函数。首先看一下Apple的帮助文档上对zip()函数的介绍。具体如下所示：

```
/// A sequence of pairs built out of two underlying sequences, where
/// the elements of the `i`th pair are the `i`th elements of each
/// underlying sequence.
public func zip<Sequence1 : SequenceType, Sequence2 : SequenceType>(sequence1: Sequence1, _ sequence2: Sequence2) -> Zip2Sequence<Sequence1, Sequence2>
```

上面那句英文的意思大概就是“基于两个基本序列构建了一个序列对，在序列对中，第i对，代表着每个基本序列中的第i个元素。”在zip函数定义的过程中，我们可以看到，zip()是一个泛型函数，其接收两个SequenceType类型的参数，然后返回一个Zip2Sequence类型的数据。新创建的序列对就存在于Zip2Sequence中。说这么多还是来个小Demo实惠一些，通过一个小实例，看zip()函数的用法一目了然。

(1) 创建两个数组zip1和zip2, 将这两个数组作为zip()函数的参数，将两个数组进行合并。具体实现如下：

```
let  zip1 = [1,2,3,5,6,7,8]
let zip2 = [10, 22, 54,56]
let zipSum = zip(zip1, zip2)
```

(2) 通过上面的程序可以看出，zipSum是一个Zip2Sequence<Array<Int>, Array<Int>>类型的常量，我们可以使用dump()对zipSum常量进行打印，观察其中的数据存储结构，具体结构如下所示：

```
dump(zipSum)
```

输出结果如下，由结果容易看出，在序列中有两个元素，第一个元素对应着数组zip1, 第二个元素对应着数组zip2。 

```
▿ Swift.Zip2Sequence<Swift.Array<Swift.Int>, Swift.Array<Swift.Int>>
  ▿ _sequences: (2 elements)
    ▿ .0: 7 elements
      - [0]: 1
      - [1]: 2
      - [2]: 3
      - [3]: 5
      - [4]: 6
      - [5]: 7
      - [6]: 8
    ▿ .1: 4 elements
      - [0]: 10
      - [1]: 22
      - [2]: 54
      - [3]: 56
```

(3)接下来就是对zipSum这个序列通过for-in循环进行遍历，下方就是对zipSum进行遍历的代码。

```
for (i0, i1) in zipSum {
    print("zip1: \(i0)-----zip2: \(i1)")
}
```

上面对zipSum遍历的结果如下所示，由下方输出结果可知，输出是成对遍历的，如果某个数组中的元素是多余的，那么就会被忽略掉。

```
zip1: 1-----zip2: 10
zip1: 2-----zip2: 22
zip1: 3-----zip2: 54
zip1: 5-----zip2: 56
```

2. 数组多个索引的延展实现

在这个将要实现的延展中，我们对Array进行了扩展，在延展中对subscript方法进行重载，使其可以接受多个下标，并且对多个下标对应的值进行索引，并把索引结果组成数组。在subscript方法中通过get方法获取索引相应的值，通过set方法为相应的索引值进行设置。下方代码段就是该延展的实现：

```
extension Array {
    subscript (i1: Int, i2: Int, rest: Int...) -> [Element] {
        get {
            var result: [Element] = [self[i1], self[i2]]
            for index in rest {
                result.append(self[index])
            }
            return result
        }
        set(values) {
            for (index, value) in zip([i1, i2] + rest, values) {
                self[index] = value
            }
        }
    }
}
```

在上述延展的实现中，并没有多少困难的地方。在subs两个cript函数中，使用的是可变参数，subscript函数参数的个数是两个以上（包括两个）。然后就是通过zip()函数以及对zip()函数返回的结果集进行遍历，从而对多个下标索引进行值的设置。经过上述延展，我们就可以通过多个索引对数组进行操作了。上述延展的使用方式如下：　

```
//MAKR: - Mutiple Subscript Array 
    func mutipleIndexArray() {
        var mutipleIndexArray = [10, 20, 30, 40, 50, 60]
        
        // get mutiple values once
        let mutipleValues = mutipleIndexArray[0, 1, 4, 5]
        print("Get Mutiple Values: \(mutipleValues)")
        
        //set mutiple values once
        mutipleIndexArray[0, 1, 5] = [100, 200, 300]
        print("Set Mutiple Values: \(mutipleIndexArray)")
        
    }
```

结果如下：

```
Get Mutiple Values: [10, 20, 50, 60]
Set Mutiple Values: [100, 200, 30, 40, 50, 300]
```

Example Extension: [GitHub](https://github.com/RobinChao/Swift-Extension)