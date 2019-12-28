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

尽管完全平衡是理想的平衡树状态，但是在现实中往往是难以实现的，要达到完全平衡的树结构，需要节点有确切的数量，才有可能达到完全平衡，因此只有一定数量的元素才能平衡。

