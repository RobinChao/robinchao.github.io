---
layout: post
author: Robin
title: Python机器学习Ch2-06 --- 大规模机器学习和随机梯度下降
tags: 机器学习 Python
categories:
  - 机器学习 
  - Python
---

到这里我们学习了如何使用梯度下降法最小化损失函数，由于梯度下降要用到所有的训练样本，因此也被称为批梯度下降。但是我们想象一下，如果我们有一个非常庞大的数据集，里面有上百万条样本，假设现在使用梯度下降法来训练模型的话，计算量是多么的惊人，每一次的求梯度都要用到所有的样本，显然不是良好的解决方案。那么能不能使用少量的样本来求梯度呢？ 答案是可以的！

**随机梯度下降法--stochastic gradient descent**由此诞生了，随机梯度下降有时也被称为迭代/在线梯度下降法。随机梯度下降法每次只使用一个样本对权重参数进行更新:

$$\eta (y^{(i)} - \phi (z^{(i)}))x^{(i)}$$

而不是基于所有样本上的累积误差综合来更新权重：

$$\Delta w = \eta \sum_i(y^{(i)} - \phi (z^{(i)}))x^{(i)}$$

和感知机算法相同。虽然随机梯度下降法被当做梯度下降的近似算法，但是实际上会比梯度下降收敛的更快，因为在相同时间内，随机梯度下降对权重的更新会更加频繁。由于单个样本得到的损失函数相对于整个训练样本得到的损失函数具有随机性，而正是这种随机性有助于随机梯度下降算法避免陷入局部最小点。在实际应用随机梯度下降法时，为了得到准确的结果，一定要以随机的方式选择样本来计算梯度，通常的做法是在每一轮迭代后将训练样本进行打乱重排（shuffle）。

> Note: 在随机梯度下降法中，通常用不断减小的自适应学习率替代固定学习率\\(\eta\\)，比如 \\(\eta = \frac {c_1}{[number of iterations] + c_2}\\)，其中\\(c_1\\)、\\(c_2\\)是常数。同时还要注意随机梯度下降并不能够保证损失函数达到局部最小点，但是结果会很接近全局最小。


随机梯度下降法的另一个有点是可以用于在线学习（online learning）。在线学习在解决不断累积的大规模数据时非常有用，比如，移动端的顾客数据。使用在线学习，系统可以实时更新并且如果存储空间不足时，可以将时间最久的数据删除，而不会影响学习。

> Note: 除了梯度下降算法和随机梯度下降算法之外，还有一种常用的折中算法，最小批学习算法（mini-batch learning）。我们知道，梯度下降算法每一次使用全量训练样本计算梯度并更新权重，随机梯度下降算法每次使用一个训练样本计算梯度更新权重，而最小批学习算法每次使用部分训练样本数据计算梯度更新权重。相对于梯度下降算法，最小批学习算法收敛速度更快，因为权重参数的更新更频繁。此外，最小批相对随机梯度下降算法，使用了向量的操作替代了for循环（每次迭代要遍历所有样本），使得计算更快。


上一节中，我们已经学习了Adaline算法，并实现了梯度下降求解Adaline，只要做部分修改，就能够得到随机梯度下降求解Adaline。

修改内容：
* fit方法修改为使用一个训练样本更新权重参数*w*
* 增加partial_fit方法，实现局部拟合
* 增加shuffle方法，打乱训练集顺序

``` python
from numpy.random import seed

# 定义Adaline类
class AdalineSGD(object):
    """自适应线性神经元分类器

    参数说明
    ------------
    eta:float
        学习率 (介于 0.0 和 1.0 之间)
    n_iter:int
        训练数据迭代次数

    属性
    -------------
    w_: 1d-array
        拟合之后的权重，1维数组.
    cost_: list
        每次迭代后的错误分类
    shuffle: bool (Default: True)
        每次迭代后打乱训练样本
    random_state: int (Default: None)
        打乱重排的状态以及初始化权重

    """
    def __init__(self, eta = 0.01, n_iter= 10, shuffle = True, random_state = None):
        self.eta = eta
        self.n_iter = n_iter
        self.w_initialized = False
        self.shuffle = shuffle
        if random_state:
            seed(random_state)
        
    
    def fit(self, X, y):
        """训练数据拟合

        参数说明
        ----------
        X : {array-like}, shape = [n_samples, n_features]
            训练集矩阵向量
            n_samples：样本数量
            n_features：特征数量
        y : array-like, shape = [n_samples]
            分类目标值

        返回值
        -------
        self : object
        """
        self._initialize_weights(X.shape[1])
        self.cost_ = []
        
        for i in range(self.n_iter):
            if self.shuffle:
                X, y = self._shuffle(X, y)
            cost = []
            for xi, target in zip(X, y):
                cost.append(self._update_weights(xi, target))
            
            avg_cost = sum(cost)/len(y)
            self.cost_.append(avg_cost)
        return self
    
    
    def partial_fit(self, X, y):
        """不更新权重对训练样本进行拟合"""
        if not self.w_initialized:
            self._initialize_weights(X.shape[1])
        if y.ravel().sjape[0] > 1:
            for xi, target in zip(X, y):
                self._update_weights(xi, target)
        else:
            self._update_weights(X, y)
        
        return self
    
    
    def _shuffle(self, X, y):
        """样本顺序打乱"""
        r = np.random.permutation(len(y))
        return X[r], y[r]
    
    
    
    def _initialize_weights(self, m):
        """初始化权重为0"""
        self.w_ = np.zeros(1 + m)
        self.w_initialized = True
        
    
    def _update_weights(self, xi, target):
        """使用Adaline学习规则更新权重"""
        output = self.net_input(xi)
        error = (target - output)
        self.w_[1:] += self.eta * xi.dot(error)
        self.w_[0] += self.eta * error
        cost = 0.5 * error ** 2
        return cost
    
    
    
    def net_input(self, X):
        """计算网络输入"""
        return np.dot(X, self.w_[1:]) + self.w_[0]
    
    def activation(self, X):
        """激活函数定义"""
        return self.net_input(X)
    
    def predict(self, X):
        """进行预测，返回类标签"""
        return np.where(self.activation(X) >= 0.0, 1, -1)
```


_shuffle方法的工作方式：调用numpy.random中的permutation函数得到0-100的一个随机序列，然后这个序列作为特征矩阵和类别向量的下标，就起到了shuffle的功能。

我们使用fit方法训练AdalineSGD模型，使用plot_decision_regions对训练结果画图:

![](/assets/adaline-sgd.png)

我们可以发现，平均损失(average cost)下降的非常快，在第15次迭代后决策界和使用梯度下降的Adaline决策界非常相似。如果我们要在在线环境下更新模型参数，通过调用partial_fit方法即可，此时参数是一个训练样本，比如ada.partial_fit(X_std[0, :], y[0])。


> 文章内容来自《Python Machine Learning》
> 
> 由于正在学习，因此在记录过程中难免有误，请不吝指正批评，谢谢！