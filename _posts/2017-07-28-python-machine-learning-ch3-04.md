---
layout: post
author: Robin
title: Python机器学习Ch3-04 --- 使用正则化解决过拟合问题
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

**过拟合(overfitting)**问题是机器学习中很常见的问题，指的是一个模型在训练集上表现很好但是其泛化能力却非常差（在测试集上表现糟糕）。如果一个模型饱受过拟合的困扰，我们也可以说此模型的方差过高，造成这个结果的原因可能是模型含有太多参数导致模型过于复杂。同样，模型也可能遇到**欠拟合(underfitting)**问题，我们也可以说此模型偏差过高，原因是模型过于简单不能学习到训练集中数据存在的模式，同样对测试集表现很差。

虽然到目前我们仅仅学习了用于分类问题的集中线性模型，过拟合和欠拟合问题可以用一个非线性决策界演示：

![](/assets/over_and_under_fitting.jpg)

怎么样才能找到bias-variance之间的平衡，常用的方法是**正则化(regularization)**。正则化是解决特征共线性、过滤数据中噪声和防止过拟合的重要方法。正则化背后的原理是引入额外的信息（偏差）来惩罚过大的权重参数。常见的形式就是所谓的**L2正则**：

$$\frac{\lambda}{2}\|w\|^2 = \frac{\lambda}{2}\sum_{j=1}^mw_j^2$$

此处的\\(\lambda\\)就是正则化系数。

> Note：正则化是特征缩放为什么重要的另一个原因。为了正则化能够其作用，需要保证所有的特征都在可比较的范围内（comparable scales）。

那么如何应用正则化呢？只需要在现有的损失函数的基础上添加正则项即可，比如逻辑回归模型，带有L2正则项的损失函数如下： 

$$J(w) = \sum_{i=1}^n\Biggl[-y^{(i)}\log\biggl(\phi\Bigl(z^{(i)}\Bigr) \biggr) - \Bigl(1 - y^{(i)}\Bigr)\log\Bigl(1 - \phi\bigl(z^{(i)}\bigr)\Bigr)\Biggr] + \frac{\lambda}{2}\|w\|^2$$
 
通过引入正则系数\\(\lambda\\)，可以控制在训练过程中参数\\(w\\)比较小。\\(\lambda\\)系数值越大，正则化的力度就越大。

现在可以理解LogisticRegression中的参数*C*，其实就是：

$$C = \frac{1}{\lambda}$$

因此，我们可以将逻辑回归正则化的损失函数重写为：

$$J(w) = C\Biggl[\sum_{i=1}^n\Biggl(-y^{(i)}\log\biggl(\phi\Bigl(z^{(i)}\Bigr) \biggr) - \Bigl(1 - y^{(i)}\Bigr)\Biggr)\log\Bigl(1 - \phi\bigl(z^{(i)}\bigr)\Bigr)\Biggr] + \frac{\lambda}{2}\|w\|^2$$

如果我们减小C的值，也就是增大正则系数\\(\lambda\\)的值，正则化的力度会变强：

![](/assets/L2.png)

上面的代码，我们训练了十个不同C值的逻辑回归模型，可以看到，随着C值的减小，权重系数也在减小。


> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！