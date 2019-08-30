---
layout: post
author: Robin
title: Python机器学习Ch2-03 --- 使用Iris数据集训练感知机模型
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

上一节，我们使用Python实现了感知机算法，这里我们来进行实际的验证。使用Iris数据集检验上一节的感知机代码，由于我们实现的是一个二分类的感知机算法，因此我们仅使用Iris数据集中的Setosa和Versicolor两种花的数据。为了简单起见，也仅仅使用sepal length和petal length两个维度的特征。但是要知道的是，感知机模型不仅仅局限于二分类问题，可以通过**One-vs-All**技巧扩展到多酚类问题上。

> **One-vs-All(OvA)**有时也被称为**One-vs-Rest(OvR)**，是一种常用的将二分类分类器扩展为多分类分类器的技巧。通过OvA技巧，我们为每一个类别训练一个分类器，此时，对应类别为正类，其余所有类别为负类。对新样本数据进行类别预测时，我们使用训练好的所有类别模型对其预测，将具有最高置信度的类别作为最后的结果。对于感知机来说，最高置信度指的是网络输入z绝对值最大的那个类别。


首先，我们使用Pandas直接从UCI读取Iris数据到**DataFrame**，然后使用Pandas的tail方法输出最后的五行数据，看一下Iris数据集的格式：

![](/assets/iris-data-tail.png)

接下来我们抽取前100条样本，正好是Setosa和Versicolor对应的样本，我们将Versicolor对应的数据作为类别**1**，Setosa对应的数据作为类别**-1**。对于特征，我们只抽取**sepal length**和**petal length**两维度特征，然后用散点图对数据进行可视化:

![](/assets/iris-data-head-100.png)

现在我们开始训练我们之前实现的感知机模型，为了更好的理解感知机训练过程，我们将每一轮的误分类为数目可视化显示出来，检查算法是否收敛以及找到分界线：

![](/assets/perceptron-practise.png)

通过上图可以发现，在第6次迭代的时候，感知机算法已经收敛了，对训练样本的预测准确率已经达到了100%。接下来，我们来绘制分割两个类别的分界线：

![](/assets/plot_decision_regions.png)

虽然对于Iris数据集，感知机算法表现的很完美，但是"收敛"一直是感知机算法中的一大问题。Frank Rosenblatt从数学上证明了只要两个类别能够被一个线性超平面分开，则感知机算法一定能够收敛。然而，如果数据并非线性可分，感知机算法则会一直运行下去，除非我们人为设置最大迭代次数n_iter，人为的设定停止条件。
