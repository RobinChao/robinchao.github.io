---
layout: post
author: Robin
title: Python机器学习Ch3-06 --- 使用松弛变量解决非线性可分问题
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

虽然上一节中我们并没有深挖SVM背后的数学概念，但是还是有必要介绍一下**松弛变量(slack variable)\\(\xi
\\)**，它是由Vladimir Vapnik在1995年引入的，借此提出了软间隔分类(soft-margin)。引入松弛变量的动机是原来的线性限制条件在面对非线性可分数据时需要松弛，这样才能保证算法收敛。

松弛变量值为正，添加到线性限制条件即可:

$$w^Tx^{(i)} \ge 1 - \xi^{(i)} \quad if \quad y^{(i)} = 1$$

$$w^Tx^{(i)} \le -1 + \xi^{(i)} \quad if \quad y^{(i)} = -1$$

新的目标函数就变成了：

$$\frac {1}{2}\|w\|^2 + C\Bigl(\sum_i\xi^{(i)}\Bigr)$$

使用变量*C*，我们可以控制错误分类的惩罚量。和逻辑回归不同，这里C越大，对于错误分类的惩罚越大。可以通过C控制间隔的宽度，在bias-variance之间找到某种平衡：

![](/assets/slack-variable.jpg)

整个概念和正则化相关，如果增加C的值就会增加bias而减小模型的方差。

下面，我们使用sklearn中内置的SVM模块训练一个模型：

![](/assets/svm_slack_var.png)

>> 关于scikit-learn中SVM的相关实现方式和使用，可参考其官网文档，这里不进行累述。

> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！