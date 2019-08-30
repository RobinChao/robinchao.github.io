---
layout: post
author: Robin
title: Python机器学习Ch2-04 --- 自适应线性神经元与学习的融合phi
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

本节我们学习另一种单层神经网络：**自适应线性神经元（ADAptive LInear NEuron，简称Adaline）**。在Frank Rosenblatt提出感知计算法不久，Bernard Widrow和他的博士生Tedd Hoff提出了Adaline算法作为感知机的改进算法(B.Widrow et al. Adaptive "Adaline" neuron using chemical "memistors".)

相对于感知机算法而言，自适应线性神经元算法更加的有趣。因为在Adaline中设计到机器学习很重要的几个概念：定义、最小化损失函数，这些概念在之后学习更加高级的算法（例如逻辑回归、SVM等）中都会被应用到，起到了抛砖引玉的作用。

**自适应线性神经元**和**感知机**的一个重要的区别是自适应线性神经元算法中权重参数更新是按照线性激活函数更新而不是单位阶跃函数。在自适应线性神经元中，激活函数如下：

$$\phi
 (z) = \phi
 (w^Tx) = w^Tx$$
 
 虽然Adaline中的激活函数更新不使用阶跃函数，但是在对测试机样本输出检测类别时还是使用阶跃函数，毕竟要输出离散的值**-1， 1**。
 
 ![](/assets/adaline-flow.png)
 
 如果我们将上图和之前的感知机算法的图解进行对比，可以发现，Adaline使用线性激活函数的连续值输出来计算模型的误差并更新权重参数，并不是二进制的类标签。
 
# 使用梯度下降算法最小化损失函数

在监督学习算法中，一个重要的概念就是定义目标函数（objective function），而目标函数就是机器学习算法在学习过程中要优化的目标，目标函数常称为损失函数（cost function），在算法学习的过程中要最小化损失函数。

对于Adaline算法，我们定义损失函数为样本真实值和预测值之间的误差平方和（Sum of Squared Errors，SSE）：

$$J(w) = \frac{1}{2}\sum_{i}(y^{(i)} - \phi (z^{(i)}))^2
$$

上面公式中的\\(\frac{1}{2}\\)完全是为了求导方便而添加的，并没有什么特殊的含义。相对于感知机中的单位阶跃函数，使用连续线性激活函数的一大优点是Adaline的损失函数是可导的。另一个很好的特性是Adaline的损失函数是凸函数，因此，我们可以更加简单有效的优化算法：**使用梯度下降（gradient descent）来找到损失函数取值最小的权重参数**。

如下图所示，我们可以把梯度下降算法看作*下山*，知道遇到局部最小点或者全局最小点才会停止计算。在每一次的迭代过程中，会沿着梯度下降的方向迈出一步，而步伐的大小则有学习率和梯度的大小共同决定。

![](/assets/gradient-descent.jpg)

使用梯度下降，实质上就是运用损失函数\\(J(w)\\)的梯度\\(\nabla J(w)\\)来对权重参数进行更新：

$$w := w + \Delta w$$

此时，权重的改变\\(\Delta w\\)的值则由负梯度乘以学习率\\(\eta	\\)确定：

$$\Delta w = -\eta\nabla J(w)$$

而要计算损失函数的梯度，我们需要计算损失函数对每一个权重参数的偏导数\\(w_j\\)：

$$\frac{\partial J}{\partial w_j} = -\sum_{i}(y^{(i)} - \phi (z^{(i)}))x_j^{(i)}
$$

因此，我们可以转换上面公式得到权重的更新\\(w_j\\)如下：

$$\Delta w_j = -\eta \frac{\partial J}{\partial w_j} = \eta \sum_{i}(y^{(i)} - \phi (z^{(i)}))x_j^{(i)}$$

注意，所有的权重参数还是同步更新的，因此Adaline算法的学习规则可以简写为：\\(w := w + \Delta w\\)。

虽然简写后的学习规则和感知机相同，但是要注意\\(\phi (z)\\)的不同。此外，还有一个很大的不同点是在计算权重更新\\(\Delta w\\)的过程中，Adaline需要用到所有训练集样本才能一次性更新所有的权重*w*，而感知机则是每次使用一个训练集样本更新所有权重参数。所以梯度下降法也被称为**批量梯度下降（batch gradient descent）**。

> Note: 详细的损失函数对权重的偏导数计算过程如下：
> 
> $$\frac{\partial J}{\partial w_j} = \frac{\partial}{\partial w_j} \frac{1}{2}\sum_i(y^{(i)} - \phi (z^{(i)}))^2 \\
> = \frac{1}{2}\frac{\partial}{\partial w_j}\sum_i(y^{(i)} - \phi (z^{(i)}))^2 \\
> = \frac{1}{2} \sum_i 2(y^{(i)} - \phi (z^{(i)}))\frac{\partial}{\partial w_j}(y^{(i)} - \phi (z^{(i)})) \\
> = \sum_i(y^{(i)} - \phi (z^{(i)}))\frac{\partial}{\partial w_j}\Biggl(y^{(i)} - \sum_i(w_j^{(i)}x_j^{(i)})\Biggr) \\
> = \sum_i(y^{(i)} - \phi (z^{(i)}))(-x_j^{(i)}) \\
> = -\sum_i(y^{(i)} - \phi(z^{(i)}))x_j^{(i)}$$





> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！