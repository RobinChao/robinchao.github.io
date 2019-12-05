---
layout: post
author: Robin
title: \#6\ Linked List 挑战
tags: 数据结构与算法
categories:
- Data Structures & Algorithms in Swift
cover: '/assets/Data-Structures-&-Algorithms-in-Swift/6/cover.jpg'
---

本文内容将针对LinkedList的五大通用性场景问题，进行分析与求解。这些问题相比多数挑战来说相对简单，主要是为了巩固关于LinkedList的知识。

## Challenge 1：创建按照反向顺序打印链表元素的函数。


```txt
// LinkedList
1 -> 2 -> 3 -> nil

// outut
3
2
1
```

 

## Challenge 2：创建返回链表中间节点值的函数。


```txt
// LinkedList
1 -> 2 -> 3 -> 4 -> nil
// middle is 3

1 -> 2 -> 3 -> nil
// middle is 2
```

## Challenge 3：创建反转链表的函数。

```txt
// LinkedList
// Before
1 -> 2 -> 3 -> nil

// After
3 -> 2 -> 1 -> nil
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

## Challenge 5：创建从链表中删除特定元素的所有匹配项的函数。

```txt
// original list
1 -> 3 -> 3 -> 3 -> 4

// list after removing all occurrences of 3
1 -> 4
```