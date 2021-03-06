---
layout: post
author: Robin
title: Python机器学习Ch2-01 --- 透过人工神经元一窥早期机器学习历史
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

# Abstract

在本章，我们要学习第一个有确定性算法描述的机器学习分类算法，感知机（perceptron）和自适应线性神经元（adaptive linear neurons）。首先我们将使用Python一步一步实现一个感知机算法，然后将其应用在鸢尾花数据集上。通过代码实现会帮助我们更好的理解分类的概念以及如何用Python有效地实现算法。学习自适应线性神经元算法以及设计到的优化知识等。

本章涉及到的知识点包括：
* 对机器学习算法有直观的了解
* 使用pandas，NumPy，matplotlib读数据、处理数据和进行数据可视化
* 使用Python实现线性分类算法

# 透过人工神经元一窥早期机器学习历史

在我们讨论感知机及其相关的算法细节前，先让我们回顾一下机器学习的早期发展历程。为了理解大脑工作原理进而设计人工智能，Warren McCullock和Walter Pitts 在1943年首次提出了一个简化版大脑细胞的概念，即McCullock-Pitts(MCP)神经元(W.S.McCulloch and W.Pitts. A Logical Calculus of the Ideas Immanent in Nervous Activity.)。神经元是大脑中内部连接的神经细胞，作用是处理和传播 化学和电信号，可见下图： 

![](/assets/human-brain.jpg)

McCullock和Pitts描述了如下的神经细胞：可以看做带有两个输出的简单逻辑门；即有多个输入传递到树突，然后在神经元内部进行输入整合，如果累积的信号量超过某个阈值，会产生一个输出信号并且通过轴突进行传递。 十几年后，基于MCP神经元模型，Frank Rosenblatt发表了第一个感知机学习规则(F.Rosenblatt, The Perceptron, a Perceiving and Recognizing Automaton. Cornell Aeronautical Laboratory, 1957)。 基于此感知机规则，Rosenblatt提出了能够自动学习最优权重参数的算法，权重即输入特征的系数。在监督学习和分类任务语境中，上面提到的算法还能够用于预测一个样本是属于类别A还是类别B。

更准确的描述是，可以将上面提到的样本属于哪一类额问题称之为二分类问题（binary classification task），我们将其中涉及到的两个类别标记为**1（正类）**和**-1（负类）**，接下来再定义一个**激活函数（activation function）\\(\phi
{(z)}\\)**，激活函数接收一个输入向量**x**和相应的权重向量**w**的线性组合，其中**z**也称为网络输入（**\\(z=w_1x_1 + ... + w_mx_m\\)**）：

$$w=\begin{bmatrix}
{w_1}\\
{w_2}\\
{\vdots}\\
{w_m}\\
\end{bmatrix}, x=\begin{bmatrix}
{x_1}\\
{x_2}\\
{\vdots}\\
{x_m}\\
\end{bmatrix}$$

此时，如果某个样本\\(x^{(i)}\\)的激活值大于事前设定的阈值，**即\\(\phi
{(z)}\\)大于阈值\\(\theta
\\)**，我们就说样本\\(x^{(i)}\\)属于类别**1**，否则属于类别**-1**。

在感知机算法中，激活函数\\(\phi
{(·)}\\)是一个简单的步进阶跃函数，有时候也称为单位阶跃函数（Heaviside step function）：

$$\phi
{(z)} = \begin{cases}
1 \quad if z\ge\theta\\
-1 \quad otherwise\\
\end{cases}$$

为了推导简单，我们可以将阈值\\(\theta
\\)挪到等式的左边，同时定义一个权重参数\\(w_0 = -\theta\\)，这样可以对*z*进行改造，使得更加的紧凑**\\(z=w_0x_0 + w_1x_1 + ... + w_mx_m = w^Tx\\)**，此时：

$$\phi
{(z)} = \begin{cases}
1 \quad if z\ge0\\
-1 \quad otherwise\\
\end{cases}$$

在下图中，左图描述了感知机的激活函数是怎样将网络输入\\(z=w^Tx\\)压缩到二元输出*（-1，1）*的，右图描述了感知机如何区分两个线性可分的类别的。

![](/assets/perceptron.jpg)

不论是MCP神经元还是Rosenblatt的阈值感知机模型，他们背后的思想都是试图使用最简单的方式来模拟人类大脑中单个神经元的工作方式：传递信号或者不传递信号。因此，Rosenblatt的最初感知机规则非常简单，步骤如下：

1. 将权重参数初始化为0或者很小的随机数
2. 对于每一个训练样本\\(x^{(i)}\\)，执行如下的步骤：
	1. 计算输出值**\\(\hat y\\)**
	2. 更新权重参数

此处的输出值就是单位阶跃函数预测的类别**(1,-1)**，参数向量*w*中的每个\\(w_j\\)的更新过程可以用数学语言表示为：

$$w_j := w_j + \Delta w_j$$

其中\\(\Delta w_j\\)用来更新权重\\(w_j\\)，在感知机算法中的计算公式为：

$$\Delta w_j = \eta (y^{(i)} - \hat y^{(i)})x_j^{(i)}$$

其中\\(\eta\\)称为学习率（learning rate），是一个介于0.0和1.0之间的常数，\\(y^{(i)}\\)是第i个训练样本的真实类别，\\(\hat y^{(i)}\\)是第i个训练样本的预测类别。**权重向量中的每个参数\\(w_j\\)**是同时被更新的。这就意味着在所有的\\(\Delta w_j\\)计算出来之前，不会重新计算\\(\hat y^{(i)}\\)。具体的，对于一个二维的数据集来说，可以将更新过程写为：

$$\Delta w_0 = \eta (y^{(i)} - output^{(i)})$$

$$\Delta w_1 = \eta (y^{(i)} - output^{(i)})x_1^{(i)}$$

$$\Delta w_2 = \eta (y^{(i)} - output^{(i)})x_2^{(i)}$$

在我们使用Python实现感知机算法之前，先来看看这个算法的美妙之处。如果感知机预测的类别是正确的，则权重参数是不做改变的，因为：

$$\Delta w_j = \eta (-1^{(i)} - -1^{(i)})x_j^{(i)} = 0$$


$$\Delta w_j = \eta (1^{(i)} - 1^{(i)})x_j^{(i)} = 0$$

当预测结果不正确的时候，权重会朝着正确的类别防线进行更新，例如当正确类别是1，权重参数就会相应的增大；否则权重参数会减小，如：

$$\Delta w_j = \eta (1^{(i)} - -1^{(i)})x_j^{(i)} = \eta (2)x_j^{(i)}$$

$$\Delta w_j = \eta (-1^{(i)} - 1^{(i)})x_j^{(i)} = \eta (-2)x_j^{(i)}$$

为了能够更好的理解，我们使用一个例子来说明。假设有如下条件：

$$y^{(i)} = +1, \hat y_j^{(i)} = -1, \eta = 1$$

我们具体化上面公式中的乘数\\(x_j^{(i)} = 0.5\\)，并且假设样本的分类结果是*-1*（进行误分类）。此时，更新权重的过程会对权重参数进行加*1*，下一次再对样本计算输出值的时候，就有更大的可能输出*1*。


$$\Delta w_j^{(i)} = (1^{(i)} - -1^{(i)})0.5^{(i)} = (2)0.5^{(i)} = 1$$

权重参数\\(\Delta w_j\\)的更新和样本\\(\x_j^{(i)}\)成正比。比如，如果我们有另一个样本\\(\x_j^{(i)} = 2\)被误分类为*-1*，在更新\\(w_j\\)的时候会朝着正确的类别*1*倾斜，并且相比较\\(x_j^{(i)} = 0.5\\)而言会倾斜更多：

$$\Delta w_j^{(i)} = (1^{(i)} - -1^{(i)})2^{(i)} = (2)2^{(i)} = 4$$

**感知机算法尽在两个类别确实线性可分并且学习率充分小的情况下才能保证收敛。**如果两个类别不能被一个决策界分开，我们可以设置最大训练集迭代次数（epochs）或者设置可容忍的错误分类样本数来停止算法的学习过程。

![](/assets/linearly-separable.jpg)

在进入下一节的代码实现之前，我们来总结一下感知机的要点：

![](/assets/perceptron-key.jpg)

感知机接收一个样本输入*x*，然后将其和权重w结合，计算网络输入*z*。*z*接着被传递给激活函数，产生一个二分类输出*-1*或*1*作为预测的样本类别。在整个学习阶段，输出用于计算预测错误率（\\(y - \hat y\\)）和更新权重参数。


> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！
