---
layout: post
author: Robin
title: Python机器学习Ch3-03 --- 通过逻辑回归对类概率建模
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

感知机算法的学习让我们了解到了分类问题，但是感知机算法具有致命的缺点：**加入数据集不能完全的线性分割，则算法永远不会收敛**。在实际的应用中，很少真正使用到感知机算法模型。

接下来我们学习一种比较流行且非常有效的线性二分类模型：**逻辑回归（logistic regression）**，可能你已经注意到了，此算法的名字中有**回归**，但是要注意的是此算法并不是回归问题的求解算法，而是百分百的分类模型。


# 逻辑回归和条件概率

逻辑回归是一个分类模型，并且易于实现，对于线性可分问题的分类效果非常好，在工业界是醉常用的恩类模型之一。和感知机以及Adaline相似，逻辑回归也是用于二分类问题的线性模型，要支持多分类问题，可以使用OvR技巧进行扩展。

逻辑回归作为一种概率模型，为了解释其背后的原理，我们先介绍几个概念：

几率(odds ratio) =  \\(\frac {p}{1 - p}\\)，其中*p*表示样本为正例的概率，这里的正例、负例的具体划分，在于实际想要预测什么，比如，我们要预测一个病人有某种疾病的概率，则病人有疾病为正例。在数学上，正例表示类别*y = 1*。有了几率的概念后，我们就可以定义**对数几率函数（logit function）**:

$$logit(p) = log\frac{p}{(1 - p)}$$

对数几率函数的自变量*p*的取值范围是[0, 1]，因变量的值域为实数域，如果将上述公式看作是类后验概率估计:

$$p(y=1|x)$$

那么定义线性关系则如下：

$$logit(p(y = 1 | x)) = w_0x_0 + w_1x_1 + ... + w_mx_m = \sum^n_{i = 0}w_mx_m = w^Tx$$

实际上，我们关心的知识某个样本属于某个类别的概率，这也正好是对数几率函数的反函数，也称为**逻辑函数**，有时候简写为**sigmoid函数**，函数图像是S型：

$$\phi(z)= \frac {1}{1 + e^{-z}}$$

其中*z*是网络输入，即权重参数和特征的线性组合\\(z = w^Tx = w_0 + w_1x_1 + ... + w_mx_m\\)。

**sigmoid函数**(S曲线)在整个机器学习中是很重要的，接下来我们来绘制**sigmoid函数**的曲线看看：

![](/assets/sigmoid_shape.png)

从上图中，我们可以看到随着网络输入*z*趋向正无穷\\(z\to \infty\\)，\\(\phi(z)\\)无限接近*1*;*z*趋向正无穷\\(z\to -\infty\\)，\\(\phi(z)\\)无限接近*0*，因此，对于sigmoid函数，其自变量的取值范围是实数域，因变量的取值范围是[0, 1]，并且**sigmoid(0) = 0.5**。

为了直观上对逻辑回归又更好的理解，我们可以将Adaline模型联系起来，二者的唯一区别是：**Adaline模型的激活函数\\((\phi(z) = z\\)在逻辑回归中，变成了sigmoid函数而已**。

![](/assets/sigmoid_func.jpg)

由于sigmoid函数的输出在[0, 1]，所以可以赋予其物理含义：样本属于正例的概率，

$$\phi(z) = P(y=1\|x;w)$$，

用Iris数据集举例说明，如果\\(\phi(z) = 0.8\\)，则意味着样本Iris-Versicoor花的概率是0.8，是Iris-Setosa花的概率是

$$P(y=0\|x;w) = 1 - P(y=1\|x;w) = 0.2$$。


有了样本的预测概率，再得到样本的类别值就简单了，和Adaline相同，可以使用单位阶跃函数：

$$\hat y = \begin{cases}
1\quad if \phi(z)\ge 0.5\\
0\quad otherwise\\
\end{cases}$$

还可以将上述公式等价于：

$$\hat y = \begin{cases}
1\quad if z\ge 0.0\\
0\quad otherwise\\
\end{cases}$$

逻辑回归之所以能够广泛的被应用，一大优点就是它不但能预测类别，还能输出具体的概率值，概率值在很多的场景下都比单纯的类别值重要得多。比如在天气预测中下雨的可能性、病人患病的可能性等等。

# 学习逻辑回归损失函数中的权重

对逻辑回归模型有了基本的了解之后，我们回到机器学习的核心问题，如何学习参数。在上一章中，定义了Adaline模型的差平方损失函数：

$$J(w) = \sum_i \frac{1}{2} (\phi(z^{(i)}) - y^{(i)})^2$$

在求解损失函数最小时的权重参数，同样，对于逻辑回归问题，我们也需要定义损失函数，在这之前，先定义**似然(likelihood)L**的概念，假设训练集中的样本是独立的，则似然的定义如下：


$$L(w) = P(y|x;w) = \prod_{i=1}^nP(y^{(i)}|x^{(i)};w) = \prod_{i=1}^n\biggl(\Bigl(\phi(z^{(i)}\Bigr)\biggr)^{y^{(i)}}\biggl(1 - \phi\Bigl(z^{(i)}\Bigr)\biggr)^{1 - y^{(i)}}$$

与损失函数尽力找到最小值相反，对于似然函数，我们要找的是最大值。实际上，对于似然函数的log值，是很容易找到最大值的，也就是最大化log-likelihood函数：


$$l(w) = \log L(w) = \sum_{i=1}^n\Biggl[y^{(i)}\log\biggl(\phi\Bigl(z^{(i)}\Bigr) \biggr) + \Bigl(1 - y^{(i)}\Bigr)\log\Bigl(1 - \phi\bigl(z^{(i)}\bigr)\Bigr)\Biggr]$$

接下来，我们可以运用梯度下降优化算法来求解最大化log-likelihood时的参数。最大化和最小化本质上没有区别，所以我们还是将log-likelihood写成求最小值的损失函数形式：

$$J(w) = \sum_{i=1}^n\Biggl[-y^{(i)}\log\biggl(\phi\Bigl(z^{(i)}\Bigr) \biggr) - \Bigl(1 - y^{(i)}\Bigr)\log\Bigl(1 - \phi\bigl(z^{(i)}\bigr)\Bigr)\Biggr]$$

为了更好的理解此损失函数，假设现在训练集只有一个样本：

$$J\biggl(\phi(z),y;w\biggr) = -y\log\biggl(\phi(z)\biggr) - (1 - y)\log\biggl(1 - \phi(z)\biggr)$$

若果上述等式右边\\(y = 0\\)，则第一项为0；如果\\(y = 1\\)，则第二项为0。如下：

$$J\biggl(\phi(z),y;w\biggr) = \begin{cases}
-\log\bigl(\phi(z)\bigr)\quad if\ y = 1\\
-\log\bigl(1 - \phi(z)\bigr)\quad if\ y = 0\\
\end{cases}$$

下图展示了一个训练样本时，不同的\\(\phi(z)\\)对应的\\(J(w)\\)：

![](\assets\varnothing.jpg)

对于上图中的蓝色线条，如果逻辑回归预测结果正确，类别则为1，损失函数值为0；对于绿色线条，如果逻辑回归预测结果正确，类别则是0，损失函数值为0。如果预测错误，则损失值趋向正无穷。

# 使用scikit-learn训练逻辑回归模型

如果我们自己实现逻辑回归，只需要将之前Adaline中的损失函数替换掉即可，对于逻辑回归，损失函数如下：

$$J(w) = -\sum_{i=1}^n\Biggl[y^{(i)}\log\biggl(\phi\Bigl(z^{(i)}\Bigr) \biggr) + \Bigl(1 - y^{(i)}\Bigr)\log\Bigl(1 - \phi\bigl(z^{(i)}\bigr)\Bigr)\Biggr]$$

不过需要考虑到sklearn中提供了高度优化过的逻辑回归实现，同时也支持多分类，因此这里不再自己实现，而是直接调用sklearn.linear_model.LogisticRegression进行逻辑回归模型训练，这里使用标准化后的Iris数据集训练模型：

![](/assets/sklearn_logistic_regression.png)

训练之后，我们画出整个决策界：

![](/assets/logistic_descion_layer.png)

可能你会发现，在初始化LogisticRegression的时候，有一个参数*C*，是什么意思呢？对于这个参数将在下一节将正则化的时候讨论。

有了逻辑回归模型，我们就可以进行预测了，如果你想要知道预测结果的概率，可以直接调用**predict_proba**方法即可：

![](/assets/logistic_regression_predict.png)

无论是Adaline还是逻辑回归，使用梯度下降算法更新权重时，用到的算式都是一样的：

$$\Delta w_j = -\eta \frac{\partial J}{\partial w_j} = \eta \sum_{i}(y^{(i)} - \phi (z^{(i)}))x_j^{(i)}$$

关于具体的推导过程，这里不再累述，有兴趣的话可以参考原书。推导的结论是逻辑回归和Adaline中的权重更新算式完全相同。

> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！