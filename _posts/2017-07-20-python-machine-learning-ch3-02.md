---
layout: post
author: Robin
title: Python机器学习Ch3-02 --- sklearn初体验之感知机模型训练
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

在前一章节中，我们学习了两个分类相关的算法：感知机和Adaline，并且使用Python进行了实现。从这节开始，我们将直接使用Python数据挖掘类库scikit-learn提供的API。


首先，我们来看看如何使用sklearn训练一个感知机模型，数据集还是使用Iris数据集，并且sklearn内置了Iris的数据集，我们只要调用对应的API即可，为了可视化方便，这里还是只使用两个维度的特征，样本数则使用三个类别全部150个样本：

![](/assets/load_iris.png)

上述代码中的unique方法，是将数据集中的文本标签转换为数字标签，更加容易使用。

为了评估训练好的模型对新数据的预测能力，我们需要把数据集分割为：训练集和测试集。

![](/assets/train-test-split.png)

这里分割数据直接使用了sklearn提供的train_tset_split方法，将数据集分割为测试集30%，训练集70%的样本。

在许多机器学习算法中，都要求对特征数据进行缩放操作，在之前的梯度下降算法中，我们也进行了数据缩放，使用的是数据标准化的过程，以及使用Python进行了实现。在sklearn中，直接提供了数据缩放的工具类StandardScaler来对特征进行标准化：

![](/assets/standard-scaler.png)

使用类StandardScaler的fit方法，StandardScaler对训练集中的**每一维特征**计算出样本平均值和标准差，然后调用transform方法对数据集进行标准化。这里的代码仅展示了对训练集进行标准化，对测试集同样要进行标准化缩放。

对数据进行标准化缩放后，就可以训练一个感知机模型了。sklearn中大多出的分类算法都支持多分类，并且默认使用了One-vs.-Rest方式实现。所以可以直接训练得到一个三分类的感知机模型。

![](/assets/multiclass-perceptron.png)

从linear_model模型中读取Perceptron类，然后初始化，使用训练集数据训练一个模型。这里的eta0就是之前章节中的学习率，n_iter表示对训练集迭代的次数，这里设置random_state参数使得shuffle结果可再现。

训练好的模型之后，我们可以使用测试集数据对其进行测试验证：

![](/assets/multiclass-predict.png)

> Note：本章中评估模型的性能仅仅依赖其在测试集上的表现。在之后的章节中，将介绍其他更多的技巧来评估模型，包括可视化分析来检测和预防过拟合（overfitting）。过拟合意味着模型对训练集中的模式捕捉很好，但是其泛化能力却很差。

最后，我们像之前一样，可视化出模型的分界区域。

![](/assets/sklearn_result_plot.png)

从输出的可视化图中可以看到，三个类别的数据并没有被线性决策面完美的分类。这也是正常的，在之前感知机部分已经介绍过，感知机对于不能够线性可分的数据，其算法的收敛是无法完成的，这也是不推荐使用感知机的原因。在接下来的章节中，将会学习到其他线性分类器，对于那些非线性可分的问题，也能够收敛到最小的损失值。


> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！phi