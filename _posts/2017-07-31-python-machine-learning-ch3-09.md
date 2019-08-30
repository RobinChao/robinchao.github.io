---
layout: post
author: Robin
title: Python机器学习Ch3-09 --- 决策树构建
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

决策树通常是将特征空间分割为矩形的方式，因此对于决策界来说还是蛮复杂的。但是要知道过大的树深度会导致过拟合问题，因此决策界并不是越复杂越好。这里，我们调用sklearn，并使用熵作为度量，训练一颗最大深度为3的决策树。还有一点，**对于决策树算法来说，特征缩放并不是必须的。**构建代码如下：

![](/assets/decision_tree.png)

从上述代码的执行结果来看，决策界和坐标轴是平行的。

sklearn的一大优点就是可以将训练好的决策树模型输出，保存在.dot文件，我们可以利用GraphViz对模型进行可视化。

我们先调用sklearn中export_graphviz将模型导出，然后利用GraphViz程序将tree.dot转为png图片：

![](/assets/export_graphviz.png)

使用如下命令转换：

``` shell
dot -Tpng tree.dot -o tree.png
```

![](/assets/tree.png)

现在，我们可以查看决策树在构建树时的过程：根节点共有105个样本，使用petal width 是否小于等于0.75分割数据为两个子节点。经过第一次分割，可以发现左子节点中的样本都是同一类型，所以停止该节点的再分割，右子节点不是同一类型，继续分割。但是需要注意的是，此时分割和根节点分割时使用的特征是不同的，使用了另一个特征。

>> 这里只是简单的对决策树进行了介绍，更加详细的使用可参考sklearn官网。

> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！