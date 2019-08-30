---
layout: post
author: Robin
title: Python机器学习Ch2-02 --- 使用Python实现感知机算法
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

在前一节中，我们学习了Rosenblatt的感知机的工作规则；接下来让我们使用Python对其进行实现，并且应用于Iris数据集。关于代码的实现，这里使用面向对象的编程思想，定义一个感知机接口作为Python类，类中的方法主要有初始化方法、fit方法和predict方法。


![](/assets/perceptron-code.png)
 
有了以上的代码实现，我们可以初始化一个新的Perceptron对象，并且对学习率eta和迭代次数n_iter赋值，fit方法先对权重参数初始化，然后对训练集中每一个样本循环，根据感知机算法对权重进行更新。类别通过predict方法进行预测。除此之外，self.errors_ 还记录了每一轮中误分类的样本数，有助于接下来我们分析感知机的训练过程。
 

> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！