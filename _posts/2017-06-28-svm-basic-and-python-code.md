---
layout: post
author: Robin
title: SVM基础知识学习与Python实践
tags: 机器学习 SVM Python
categories:
  - 机器学习 
  - SVM
  - Python
---

![](/assets/SVM_BLOG_COVER.jpg)

## 简介

在机器学习中，数据的分类是一种非常重要的事务。支持向量机(SVM)在模式分类和非线性回归领域中有着广泛的应用。SVM算法的原始形式由Vladimir N. Vapnik和Alexey Ya提出。 此后，支持向量机得到巨大变革，成功应用于诸如文本（和超文本）分类，图像分类，生物信息学（蛋白质分类，癌症分类），手写字符识别等许多现实问题中。

## 目录

* 什么是支持向量机？
* SVM是如何工作的？
* SVM求导方程
* SVM的优缺点
* Python中的SVM

## 什么是支持向量机？

支持向量机是一种监督机器学习算法，可用于分类和回归问题中。它遵循称为内核技巧的技术来转换数据，并且基于这些转换，找到可能的输出之间的最佳边界。

简单来说，它做了一些非常复杂的数据转换，以确定如何根据定义的标签或输出来分离数据。在本文中，我们将仅学习SVM分类算法。

![](/assets/classification.jpg)

## SVM是如何工作的？

SVM的主要思想是确定最大化训练数据边缘的最优分离超平面。让我们通过术语来理解这个客观的术语。

### 什么是分离超平面？

首先，我们可以看上部分中给出的图片，思考如何区分两种不同颜色的数据？例如，我们可以画一条线，将红色部分和绿色部分分割开来，而这样的线就叫做分离超平面。

但是会有一个问题，为什么把一条线叫做超平面呢？

在上图中，我们仅仅考虑了最简单的数据样本，数据均位于一个二维的平面<img src="http://latex.codecogs.com/gif.latex?R^{2}" title="R^{2}" />中。但是支持向量机也支持一般的`n维`数据样本。而在高维在情况下，超平面就会使一个平面。

更加正式的解释是，它是一个`它是n维欧几里德空间的n-1维子空间`。因此对于

* 1维数据：点代表超平面。
* 2维数据：线代表超平面。
* 3维数据：面代表超平面。
* 更高维数据：它被称为超平面。

我们已经知道SVM的目标是找到最优分离超平面。那么什么样的分离超平面是最优的呢？

但是在实践中，即使存在一个分离数据集的超平面，也并不意味着这个超平面就是最优的。

让我们看一下分离数据的实验图，来进一步理解最优分离超平面。

1. 多个超平面

在下图中，有多个超平面，但是那个才是最优的超平面呢？很容易看到分割线B能够最好的额分离两个类的数据样本，因此分割线B是此种情况下的最优超平面。

![](/assets/multple-hyperplanes.jpg)

2. 多个分离超平面

也可能会有多个分离超平面，如下图。那么此种情况如何找到最优的分离超平面呢？直观的，我们可能会选择一条距离某一个数据类比较近的分割线作为此种情况下的超平面，例如下图中的分割线C，但是，当分割线逆时针有一点的偏移，具体分割线C的红色数据样本就可能落在分割线C上，甚至越过分割线C，给最终的分类造成误差和错误。因此，最终我们选择尽可能远离每个数据样本类的分割线作为此种情况下的分离超平面。

![](/assets/multiple-separating-hyperplanes.jpg)

对于上图此种情况，满足最优分离超平面的分割线则是分割线B。

由上可知，超平面的特点是最大化每个类的最近点和超平面之间的距离，从而获得超平面，而这个距离被称为`边距`，如下图。

![](/assets/Margin.png)

## SVM求导方程

### 数据设置

现在，我们已经了解了SVM算法的基础知识，回顾SVM的目标 --- 找到最优分离超平面，而如何去寻找，除了上述例子中直观的方式外，还有SVM所使用的数据技术，接下来让我们来看看SVM中寻找最优超平面的数据技术。

在开始之前，你可能还需要了解如下数据概念：向量，向量运算（加法，减法，点积）和正交投影等。

### 超平面方程

在探寻超平面的方程之前，先来看看直线的方程：

<img src="http://latex.codecogs.com/gif.latex?y=mx&plus;c" title="y=mx+c" /> 

*其中，m表示直线的倾斜程度，即斜率，y表示直线在y轴上的截距。*

而超平面的广义方程如下：

<img src="http://latex.codecogs.com/gif.latex?w^Tx=0" title="w^Tx=0" />

这里的*w*和*x*是向量，<img src="http://latex.codecogs.com/gif.latex?w^Tx" title="w^Tx" />表示两个向量之间的点积。*w*向量通常被称为**权重向量**。

假设我们将线性方程<img src="http://latex.codecogs.com/gif.latex?y=mx&plus;c" title="y=mx+c" />转换一下：

<img src="http://latex.codecogs.com/gif.latex?y-mx-c=0" title="y-mx-c=0" />

在这种情况下，*w*和*x*向量的取值范围如下：

<img src="http://latex.codecogs.com/gif.latex?w=\begin{pmatrix}-c\\-m\\1&space;\end{pmatrix}" title="w=\begin{pmatrix}-c\\-m\\1 \end{pmatrix}" /> ，  <img src="http://latex.codecogs.com/gif.latex?x=\begin{pmatrix}1\\x\\y&space;\end{pmatrix}" title="x=\begin{pmatrix}1\\x\\y \end{pmatrix}" />

到这里你可能已经发现，将向量*w*和*x*进行点积之后，就会得到线性方程<img src="http://latex.codecogs.com/gif.latex?y-mx-c=0" title="y-mx-c=0" />，而使用方程<img src="http://latex.codecogs.com/gif.latex?w^Tx=0" title="w^Tx=0" />表示，仅仅是其另外一种表达方式而已，两者表示的是相同的东西。

但是通常情况下，为什么要使用<img src="http://latex.codecogs.com/gif.latex?w^Tx=0" title="w^Tx=0" />呢？简单点而说，因为在更好维度数据集的情况下，这种方式能够更加容易的处理，并且*w*表示的是垂直于超平面的向量。一旦开始计算点到超平面的距离时，这个属性将会非常的有用。

### 理解约束

在分类问题中，训练数据通常类似如下的形式：

<img src="http://latex.codecogs.com/gif.latex?\{(x_1,y_1),(x_2,y_2),...,(x_n,y_n)\}&space;\in&space;\mathbb{R}^n&space;\times&space;{-1,1}" title="\{(x_1,y_1),(x_2,y_2),...,(x_n,y_n)\} \in \mathbb{R}^n \times {-1,1}" />

这意味着训练数据集是<img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />的一对值，一个n维向量的特征和<img src="http://latex.codecogs.com/gif.latex?y_i" title="y_i" />，以及标签<img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />。当<img src="http://latex.codecogs.com/gif.latex?y_i=1" title="y_i=1" />时，表示特征向量<img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />属于类1，当<img src="http://latex.codecogs.com/gif.latex?y_i=-1" title="y_i=-1" />，表明属于类-1。

在分类问题中，我们尝试找出一个函数，<img src="http://latex.codecogs.com/gif.latex?y=f(x):&space;\mathbb{R}^n&space;\longrightarrow&space;\{-1,1\}" title="y=f(x): \mathbb{R}^n \longrightarrow \{-1,1\}" />，<img src="http://latex.codecogs.com/gif.latex?f(x)" title="f(x)" />是从训练数据集中学习得到的，然后将它的知识应用到未知的数据中去。

然而，<img src="http://latex.codecogs.com/gif.latex?f(x)" title="f(x)" />函数的数量是无限的，我们必修限制我们处理数据的函数的类，在SVM中，<img src="http://latex.codecogs.com/gif.latex?w^Tx=0" title="w^Tx=0" />函数的功能是表示为超平面的功能。

它也可以被表示为： <img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b=0;&space;\vec{w}\in&space;\mathbb{R}^n&space;\mbox{&space;and&space;}&space;b&space;\in&space;\mathbb{R}" title="\vec{w}.\vec{x}+b=0; \vec{w}\in \mathbb{R}^n \mbox{ and } b \in \mathbb{R}" />

这就将输入空间分为两部分，一部分是包含类 1 的向量，另一部分是包含类 -1 的向量。

在本文接下来的部分中，我们将考虑二维向量。让<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_0" title="\mathcal{H}_0" />表示一个超平面分离数据集并满足以下条件：

在选择超平面<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_0" title="\mathcal{H}_0" />的同时，我们同时选择另外两个超平面<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_1" title="\mathcal{H}_1" />和<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_2" title="\mathcal{H}_2" />，并且他们同样能够分离数据，并满足如下条件：

<img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b=\delta" title="\vec{w}.\vec{x}+b=\delta" />  

<img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b=\mbox{-}\delta" title="\vec{w}.\vec{x}+b=\mbox{-}\delta" />

这使得<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_0" title="\mathcal{H}_0" />以及<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_1" title="\mathcal{H}_1" />和<img src="http://latex.codecogs.com/gif.latex?\mathcal{H}_2" title="\mathcal{H}_2" />等距离。

变量δ不是必须的，所以我们可以设置 * δ = 1* 来简化问题，简化后的方程如下：

<img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b=1" title="\vec{w}.\vec{x}+b=1" />

<img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b=-1" title="\vec{w}.\vec{x}+b=-1" />

接下来我们还要确认这两个超片面之间没有数据点，因此，我们将仅选择满足以下限制的那些超平面：

对于每一个向量<img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />：

1. <img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />具包含类 -1 ：<img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b\leq&space;\mbox{-}1" title="\vec{w}.\vec{x}+b\leq \mbox{-}1" />  或者
2. <img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b\geq&space;1" title="\vec{w}.\vec{x}+b\geq 1" />，<img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />具包含类 1。

![](/assets/constraints.png)

### 组合约束

上述的约束都可以组合成为单个约束。

#### 约束1

对于包含类 -1 的向量 <img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />，<img src="http://latex.codecogs.com/gif.latex?\vec{w}.\vec{x}&plus;b\leq&space;\mbox{-}1" title="\vec{w}.\vec{x}+b\leq \mbox{-}1" />

在两侧同时乘以<img src="http://latex.codecogs.com/gif.latex?y_i" title="y_i" />（这个问题中，始终为-1），得到：

<img src="http://latex.codecogs.com/gif.latex?y_i\left(\vec{w}.\vec{x}&plus;b\right)\geq&space;y_i(-1)" title="y_i\left(\vec{w}.\vec{x}+b\right)\geq y_i(-1)" />

意味着<img src="http://latex.codecogs.com/gif.latex?y_i\left(\vec{w}.\vec{x}&plus;b\right)&space;\geq&space;1" title="y_i\left(\vec{w}.\vec{x}+b\right) \geq 1" />对于向量 <img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />始终包含类-1。

#### 约束2： <img src="http://latex.codecogs.com/gif.latex?y_i=1" title="y_i=1" />

<img src="http://latex.codecogs.com/gif.latex?y_i\left(\vec{w}.\vec{x}&plus;b\right)&space;\geq&space;1" title="y_i\left(\vec{w}.\vec{x}+b\right) \geq 1" />对于向量 <img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />始终包含类1。

综合上述方程，我们可以得到：

<img src="http://latex.codecogs.com/gif.latex?y_i\left(\vec{w}.\vec{x}&plus;b\right)\geq&space;1&space;\mbox{&space;for&space;all&space;}1\leq&space;i\leq&space;n" title="y_i\left(\vec{w}.\vec{x}+b\right)\geq 1 \mbox{ for all }1\leq i\leq n" />

这导致唯一的约束，这两个约束在数学上是等价得。组合的新约束也具有相同的效果，即，两个超平面之间没有点。

### 最大化边距

为了简单起见，这里直接跳过计算边距的公式推导，即边距*m*：

<img src="http://latex.codecogs.com/gif.latex?\displaystyle&space;m=\frac{2}{||\vec{w}||}" title="\displaystyle m=\frac{2}{||\vec{w}||}" />

这个公式中唯一的变量是*w*,它与*m*是成反比例的。因此最大化边距问题就演变成了最小化<img src="http://latex.codecogs.com/gif.latex?||\vec{w}||" title="||\vec{w}||" />了。这就导致了如下的优化问题：

在<img src="http://latex.codecogs.com/gif.latex?y_i\left(\vec{w}.\vec{x}&plus;b\right)&space;\geq&space;1&space;\mbox{&space;for&space;any&space;}&space;i=1,\dots,&space;n" title="y_i\left(\vec{w}.\vec{x}+b\right) \geq 1 \mbox{ for any } i=1,\dots, n" />的条件下最小化<img src="http://latex.codecogs.com/gif.latex?\displaystyle&space;(\vec{w},b)&space;\{&space;\frac{||\vec{w}||^2}{2}" title="\displaystyle (\vec{w},b) \{ \frac{||\vec{w}||^2}{2}" />。

上面是是数据线性可分的情况下。在许多情况下，数据可能并不是线性可分的，此时，支持向量机将查找使得边距最大化并最大限度减少错误分类的超平面。

因此支持向量机引入了`松弛变量` <img src="http://latex.codecogs.com/gif.latex?\zeta_i" title="\zeta_i" />，它允许一些数据对象可以落在边缘甚至另一类，但是这些数据点有着惩罚值。

![](/assets/Slack-variables.png)

在这种情况下，算法尝试保持松弛变量为零，同时最大化边距。然而，它将误分类与边缘超平面的距离之和最小化，而不是错误分类的数量。

我们将约束修改如下：

<img src="http://latex.codecogs.com/gif.latex?y_i(\vec{w}.\vec{x_i}&plus;b)\geq&space;1-\zeta_i&space;\mbox{&space;for&space;all&space;}&space;1\leq&space;i\leq&space;n,&space;\zeta_i\geq&space;0" title="y_i(\vec{w}.\vec{x_i}+b)\geq 1-\zeta_i \mbox{ for all } 1\leq i\leq n, \zeta_i\geq 0" />

最终优化问题也就变为了：

在<img src="http://latex.codecogs.com/gif.latex?y_i\left(\vec{w}.\vec{x}&plus;b\right)&space;\geq&space;1-\zeta_i&space;\mbox{&space;for&space;any&space;}&space;i=1,\dots,&space;n" title="y_i\left(\vec{w}.\vec{x}+b\right) \geq 1-\zeta_i \mbox{ for any } i=1,\dots, n" />的条件下最小化<img src="http://latex.codecogs.com/gif.latex?\displaystyle&space;(\vec{w},b)&space;\{&space;\frac{||\vec{w}||^2}{2}&plus;C\sum_i\zeta_i" title="\displaystyle (\vec{w},b) \{ \frac{||\vec{w}||^2}{2}+C\sum_i\zeta_i" />的问题。

这里，参数*C*是控制在松弛变量惩罚分（误分类）和边际宽度之间权衡的正则化参数。

* 小的*C*会使得约束很容易忽略，从而产生一个大的边距；
* 大的*C*会是得约束条件很难忽略，从而导致一个小的边距；
* 对于<img src="http://latex.codecogs.com/gif.latex?C=\inf" title="C=\inf" />，所有的约束都被增强。

在2D数据的情况下，两类数据的超平面是线，在3D数据情况下，分离数据的超平面是平面，但是并不总可能使用线或者平面，有时候需要一个非线性区域来分离这些类。对于这种情况，支持向量机通过使用`内核函数`来处理，该函数将数据映射到不同的空间，其中，线性超平面可以分离类。这种技巧被称为`核技巧`，其中内核函数将数据变换到更高维度的特征空间，使得线性可分是可能的。

![](/assets/kernel.png)

如果<img src="http://latex.codecogs.com/gif.latex?\phi" title="\phi" />是<img src="http://latex.codecogs.com/gif.latex?x_i" title="x_i" />映射到<img src="http://latex.codecogs.com/gif.latex?\phi(x_i)" title="\phi(x_i)" />的核函数，则约束就变为：

<img src="http://latex.codecogs.com/gif.latex?y_i(\vec{w}.\phi(x_i)&plus;b)\geq&space;1-\zeta_i&space;\mbox{&space;for&space;all&space;}&space;1\leq&space;i&space;\leq&space;n,&space;\zeta_i\geq&space;0" title="y_i(\vec{w}.\phi(x_i)+b)\geq 1-\zeta_i \mbox{ for all } 1\leq i \leq n, \zeta_i\geq 0" />

则最优化问题就变为：

在<img src="http://latex.codecogs.com/gif.latex?y_i(\vec{w}.\phi(x_i)&plus;b)\geq&space;1-\zeta_i&space;\mbox{&space;for&space;all&space;}&space;1\leq&space;i&space;\leq&space;n,&space;\zeta_i\geq&space;0" title="y_i(\vec{w}.\phi(x_i)+b)\geq 1-\zeta_i \mbox{ for all } 1\leq i \leq n, \zeta_i\geq 0" />的条件下最小化<img src="http://latex.codecogs.com/gif.latex?\displaystyle&space;(\vec{w},b)&space;\{&space;\frac{||\vec{w}||^2}{2}&space;&plus;&space;C\sum_i\zeta_i" title="\displaystyle (\vec{w},b) \{ \frac{||\vec{w}||^2}{2} + C\sum_i\zeta_i" />的问题。

用于解决这些优化问题的最常用的方法是凸优化，这里不会深入这些优化问题的解决方案。

## SVM的优缺点

每个分类算法都有其自己的优缺点，他们根据正在分析的数据集的特点起着不同的作用。

SVM的一些优点如下：

* 凸度优化方法的性质确保了最优性。该解决方案保证是全局最小值，而不是局部最小值。
* SVM是一种适用于线性和非线性可分离数据（使用内核技巧）的算法。唯一要做的就是找出恰当的正则化项*C*；
* SVM在小和高维数据空间上效果很好。它对于高维数据集很有效，因为SVM中的训练数据集的复杂度通常由支持向量的数量而不是维度来表征。即使删除所有其他训练示例并重复训练，我们将获得相同的最佳分离超平面；
* SVM可以在较小的训练数据集上有效工作，因为它们不依赖于整个数据；

SVM的一些缺点如下：

* 它们不适合较大的数据集，因为使用SVM的训练时间可能很高，并且计算量更大；
* 它们在具有重叠类的噪声数据集上效果很差； 

## Python中的SVM

我们来看看在Python中用来实现SVM的库和函数。

在Python中最广泛使用的，实现机器学习算法的库是[scikit-learn](http://scikit-learn.org/)。scikit-learn中用于SVM分类的类是*svm.SVC()*:

``` python
sklearn.svm.SVC (C=1.0, kernel='rbf', degree=3, gamma='auto')
```

其中的参数如下：

* **C**：它是误差项的正则化参数
* **kernel**：它指定要在算法中使用的内核类型。它可以是`linear`、`poly`、`rbf`、`sigmoid`、`precomputed`或一个可调用类型。默认值为`rbf`。
* **degree**：它是多项式核函数`poly `的程度，并被所有其他内核忽略。默认值为3。
* **gamma**：它是`rbf`、`poly`和`sigmoid`的核系数。如果gamma是`auto`，则将使用***1/n_features*。

还有很多更高级的参数，这里不再一一列举，可以参考[官网](http://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html#sklearn.svm.SVC)中的说明。

下面是使用Python演示如何使用SVM解决二元分类问题：

``` python
from sklearn import datasets, svm
from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix


def load_data():
    iris = datasets.load_iris()
    X = iris.data[:, :2]
    y = iris.target
    return X, y


def split_train_test_data(X, y):
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=0)
    return X_train, X_test, y_train, y_test

def train_linear_kernel():
    X, y = load_data()
    X_train, X_test, y_train, y_test = split_train_test_data(X, y)
    svc_linear = svm.SVC(kernel='linear', C=1)
    svc_linear.fit(X_train, y_train)
    predicted = svc_linear.predict(X_test)
    cnf_matrix = confusion_matrix(y_test, predicted)
    print cnf_matrix


if __name__ == '__main__':
    train_linear_kernel()
```

运行后，输入结果为：

```
[[16  0  0]
 [ 0 13  5]
 [ 0  4  7]]
 ```
 
可以通过更改参数<img src="http://latex.codecogs.com/gif.latex?C,&space;\gamma" title="C, \gamma" />和内核函数来调整SVM 。调整scikit-learn中可用参数的函数称为*gridSearchCV()*。

``` python
sklearn.model_selection.GridSearchCV(estimator, param_grid)
```

此函数的参数说明：

* **estimator**:它是estimator对象，在上述例子中是*svm.SVC()*。
* **param_grid**:它是具有参数名称（字符串）作为键的字典或列表，以及作为值的参数设置列表。

想要了解更多*gridSearchCV()*函数的参数，可参考[官网](http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html#)说明。

示例代码如下：

``` python
from sklearn import datasets, svm
from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix
from sklearn.model_selection import GridSearchCV


def load_data():
    iris = datasets.load_iris()
    X = iris.data[:, :2]
    y = iris.target
    return X, y


def split_train_test_data(X, y):
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=0)
    return X_train, X_test, y_train, y_test

def grid_search_cv_parameters():
    return {'kernel':('linear', 'rbf'), 'C':[1,2,3,4,5,6,7,8,9,10], 'gamma': 
              [0.01,0.02,0.03,0.04,0.05,0.10,0.2,0.3,0.4,0.5]}


def train_linear_kernel():
    X, y = load_data()
    X_train, X_test, y_train, y_test = split_train_test_data(X, y)
    parameters = grid_search_cv_parameters()
    svr = svm.SVC()
    grid = GridSearchCV(svr, parameters)
    grid.fit(X_train, y_train)
    predicted = grid.predict(X_test)
    cnf_matrix = confusion_matrix(y_test, predicted)
    print(cnf_matrix)


if __name__ == '__main__':
    train_linear_kernel()
```

输出结果如下：

```
[[16  0  0]
 [ 0 13  5]
 [ 0  3  8]]
```

在上面的代码中，我们考虑调整的参数是`内核`，`C`和`gamma`。从中得到最佳值的值并写在括号中。这里，只给出了部分值，如果需要给出全部的数值，需要执行更长的时间来获取，这里不再演示。

## 总结

在本文中，对SVM分类算法进行了一个非常基础的解释。这里略过了一些数学困难，如计算距离和解决优化问题。希望这给了你足够的知识，来了解机器学习算法SVM是如何根据提供的数据集类型来进行修改的。

在本文中，对SVM分类算法进行了基本的解释。 已经排除了一些数学上的复杂性，如计算距离和解决优化问题。 但是，希望这能给您足够的基础知识，了解如何根据提供的数据集的类型使用和优化机器学习算法 --- SVM。



