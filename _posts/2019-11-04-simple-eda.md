---
layout: post
author: Robin
title: 探索性数据分析入门
tags: 数据科学 Python
categories:
- 数据科学
cover: '/assets/simple-eda/cover.png'
---

在数据科学领域里，最具挑战的问题之一便是如何确定数据对特定问题带来价值。在使用机器学习或者深度学习之前，确定数据或者特征是否利于特定问题，是数据科学后续工作的重中之重。

因此，在进行数据科学问题之前，通常会对数据进行分析，洞察数据中所涵盖的深层特性是否利于特定问题，以及是否适用于所选用的机器学习算法等，而这一步被称为**探索性数据分析（Exploratory Data Analysis， EDA）**。

**探索性数据分析（Exploratory Data Analysis，EDA）** 是指对已有数据在尽量少的先验假设下通过作图、制表、方程拟合、计算特征量等手段探索数据的结构和规律的一种数据分析方法，该方法在上世纪70年代由美国统计学家J.K.Tukey提出。传统的统计分析方法常常先假设数据符合一种统计模型，然后依据数据样本来估计模型的一些参数及统计量，以此了解数据的特征，但实际中往往有很多数据并不符合假设的统计模型分布，这导致数据分析结果不理想。EDA则是一种更加贴合实际情况的分析方法，它强调让数据自身“说话”，通过EDA我们可以最真实、直接的观察到数据的结构及特征。

> **探索性数据分析（EDA）是一种数据分析方法，它采用多种技术来最大化对数据集的洞察力。揭示底层结构；提取重要变量；检测异常值和异常；建立简约模型；并确定最佳因子设置。**

![](/assets/simple-eda/banner.png)

## 从实战中学习EDA

**实践是检验整理的唯一途径。** 为了能够更好更快的理解EDA，这里将直接从**[鸢尾花数据集(UCI Machine Learning Repository)](https://archive.ics.uci.edu/ml/datasets/Iris)**的探索性分析中学习EDA的方法。

**目标：** 从给定4个维度特征的鸢尾花数据集中学习，已确定新的鸢尾花属于3个鸢尾花类别中的哪一个类别。

> **在进行EDA的过程中，需要始终牢记最初确立的目标，否则EDA可能偏离目标！**

### 导入所需类库

显而易见，进行Python语言相关的开发时，第一步基本上都是导入所需的类库（前提是类库已经被安装在当前坏境）。在进行EDA时，所需的类库可能并不是很多，满足需求即可，在这里，将导入像*Pandas、Matplotlib、numpy*等类库，对应类库的作用，可自行搜索学习。


```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import os
```

### 下载数据并加载


```python
'''downlaod iris.csv from https://raw.githubusercontent.com/uiuc-cse/data-fa14/gh-pages/data/iris.csv'''
#Load Iris.csv into a pandas dataFrame.
iris = pd.read_csv("./iris.csv")
iris.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sepal_length</th>
      <th>sepal_width</th>
      <th>petal_length</th>
      <th>petal_width</th>
      <th>species</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.1</td>
      <td>3.5</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.9</td>
      <td>3.0</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.7</td>
      <td>3.2</td>
      <td>1.3</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.6</td>
      <td>3.1</td>
      <td>1.5</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5.0</td>
      <td>3.6</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
  </tbody>
</table>
</div>



* *.head()*函数是Pandas中的标准函数，用于观察数据集的数据详情，默认情况下返回数据集的前5个样本点。同时，*.tail()*函数返回数据集的后5个样本点。


```python
# Data-points and features
iris.shape
```




    (150, 5)



*.shape*参数可以查看数据集的形状（行数、列数）。

* 此处使用的鸢尾花数据集是一个150行和5列的数据集。


```python
iris.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 150 entries, 0 to 149
    Data columns (total 5 columns):
    sepal_length    150 non-null float64
    sepal_width     150 non-null float64
    petal_length    150 non-null float64
    petal_width     150 non-null float64
    species         150 non-null object
    dtypes: float64(4), object(1)
    memory usage: 6.0+ KB


*.info()*函数用于展示数据集列数据的数据类型情况。

* 此处，数据只有float类型和object类型两种值类型数据；
* 无变量或列包含null值或者缺失值。


```python
iris.columns
```




    Index(['sepal_length', 'sepal_width', 'petal_length', 'petal_width',
           'species'],
          dtype='object')



* *.columns* 用来查看数据集的列或特征的名称。


```python
iris['species'].value_counts()
```




    versicolor    50
    setosa        50
    virginica     50
    Name: species, dtype: int64



* *.value_counts()*是对数据集上特定列进行降序后，获取该列的每个值的计数值；
* 此处，每一个种类（Versicolor, Setosa, Virginica）各有50个观察对象，因此该数据集应该是**均匀分布**的。


```python
iris.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sepal_length</th>
      <th>sepal_width</th>
      <th>petal_length</th>
      <th>petal_width</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>150.000000</td>
      <td>150.000000</td>
      <td>150.000000</td>
      <td>150.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.843333</td>
      <td>3.054000</td>
      <td>3.758667</td>
      <td>1.198667</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.828066</td>
      <td>0.433594</td>
      <td>1.764420</td>
      <td>0.763161</td>
    </tr>
    <tr>
      <th>min</th>
      <td>4.300000</td>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>0.100000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5.100000</td>
      <td>2.800000</td>
      <td>1.600000</td>
      <td>0.300000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>5.800000</td>
      <td>3.000000</td>
      <td>4.350000</td>
      <td>1.300000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>6.400000</td>
      <td>3.300000</td>
      <td>5.100000</td>
      <td>1.800000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>7.900000</td>
      <td>4.400000</td>
      <td>6.900000</td>
      <td>2.500000</td>
    </tr>
  </tbody>
</table>
</div>



* *.describe()*函数用于获取数据集的各种汇总统计信息。该函数返回计数值、均值、标准差、最小值和最大值、以及数据的分位数。

至此，对数据集来说已经有了一个基础的了解，对于数据的探索性分析来说，这才刚刚开始，往往通过对数据的图形化描述能够更加详细的了解数据特征之间的关系等，包括单变量和多变量分析等。

## 二维散点图

散点图是一种将数据显示为点集合的图。点的位置取决于其二维值，每个值都是水平或者垂直维度上的位置。


```python
# 2-D Scatter plot with color-coding for each flower type/class.
# Here 'sns' corresponds to seaborn
sns.set_style('whitegrid')
sns.FacetGrid(iris, hue='species', height=8) \
    .map(plt.scatter, 'sepal_length', 'sepal_width') \
    .add_legend()
plt.show()
```


![png](/assets/simple-eda/output_21_0.png)


Seaborn中的*FacetGrid*类有助于使用多个面板在数据集的子集中可视化一个变量的分布以及多个变量之间的关系。参数*hue*根据与每个参数相关的颜色将数据点分开。

三个类别的数据点根据*sepal_length*分散。

* 使用*sepal_length*和*sepal_width*特征，可以区分Setosa同其他类别（线性可分）；
* 区分Versicolor和Virginica相对困难一点，因为它们之间有大量的重叠部分。

## 对图

对图有助于查看数据中单个变量的分布以及多个变量之间的关系。


```python
# pairwise scatter plot: Pair-Plot
# Dis-advantages: Cannot visualize higher dimensional patterns in 3-D and 4-D.
# Only possible to view 2D patterns.
plt.close()
sns.pairplot(iris, hue='species', size=3)
plt.show()
```


![png](/assets/simple-eda/output_24_0.png)


* *petal_length*和*petal_width*是区分各种类型鸢尾花的重要特征；
* Setosa类型的鸢尾花很容易被识别（线性可分），Virginica和Versicolor的特征有一些重叠（接近于线性可分）；
* 可以找到“分割线”或者“if-else”条件来建立一个简单的模型，对鸢尾花的类型进行分类。

## 直方图和PDF（概率密度函数）


```python
sns.FacetGrid(iris, hue='species', height=8) \
    .map(sns.distplot, 'petal_length') \
    .add_legend()
plt.show()
```


![png](/assets/simple-eda/output_27_0.png)


*distplot()* 函数绘制了各种鸢尾花类型的*petal_length*的分布。图中蓝色代表Setosa类型鸢尾花的*petal_length*的直方图，黄色、绿色类似。y轴代表x轴上一个小窗口或者一个间隔中存在的数据点的数量，意味着在x轴上给定一个点或者区域，该点或者区域上直方图的高度代表x轴上该点或者区域有多少数据点。在上图中小窗口定义为*petal_length*。

直方图的高度越高，即在给定的区域内密度越大，则找到的种类和花瓣的长度值之间的对应越多。因此上图也称为**概率密度图**，通过对直方图进行平滑处理（KDE）制成的图像曲线为PDF，即概率密度函数曲线。

**结论：**

* 如果 `petal_length ≤ 2`，种类为 Setosa；
* 如果 `petal_length ＞ 2`，并且 `petal_length ≤ 4.7`，种类为 Versicolor；
* 如果 `petal_length ≥ 4.7`，种类为 Virginica；
* 另一个结论是，通过 `petal_length` 单变量的分析，对于区分不同的鸢尾花种类很有帮助，仅仅使用这一个特征，可以构建一个使用if-else条件判定的简单模型。

> 在区分Versicolor的时候，使用了4.7作为分界点，而不是5的原因是， `petal_length ≤ 4.7`条件下，分类结果更多的可能性是Versicolor，而不是Virginica，这也和数据可视化的结果更为接近。

当然，也可以使用*petal_width*、*sepal_length*、*sepal_width*进行单变量的分析，但是最终的结果可能并没有使用*petal_length*的结果好。

PDF的局限性在于，无法查看其直观的图标或者统计性数据。例如，无法根据*petal_length*单变量分析，看到`petal_length ＜ 5`的情况下，属于Versicolor类型数据的百分比等。

鉴于此，还需要使用CDF（累积分布函数）。

## CDF（累积分布函数）

累积分布函数计算给定x值的累积概率。可以使用CDF来确定从总体中抽取的随机观察值小于或者等于某个值的可能性。

CDF的优势在于可以通过可视化的方式查看，例如查看Setosa类型的鸢尾花，*petal_length* 小于1.6的百分比。PDF和直方图无法提供相同的确切的百分比，PDF只是分布图。


```python
# Need for Cumulative Distribution Function (CDF)
# We can visually see what percentage of setosa flowers have a
# petal_length of less than 1.6

# Plot CDF petal_length
iris_setosa = iris.loc[iris['species'] == 'setosa']
iris_virginica = iris.loc[iris['species'] == 'virginica']
iris_versicolor = iris.loc[iris['species'] == 'versicolor']

counts, bin_edges = np.histogram(iris_setosa['petal_length'], bins=10, density=True)

pdf = counts/(sum(counts))
print(pdf)
print(bin_edges)

# compute CDF
cdf = np.cumsum(pdf)
plt.plot(bin_edges[1:], pdf, label='PDF')
plt.plot(bin_edges[1:], cdf, label='CDF')

plt.legend()
plt.show()
```

    [0.02 0.02 0.04 0.14 0.24 0.28 0.14 0.08 0.   0.04]
    [1.   1.09 1.18 1.27 1.36 1.45 1.54 1.63 1.72 1.81 1.9 ]



![png](/assets/simple-eda/output_30_1.png)


示例代码中构建了三个数据框对应三种不同的鸢尾花种类。图中x轴代表*petal_length*，y轴则是对应的累积分布概率。

`cumsum()`函数是NumPy类库中通过PDF计算CDF的方法。

* 假设`petal_length`的值我们关心的是1.6。对于1.6，数据中有接近82%的Setosa类型鸢尾花，`petal_length ≤ 1.6`。即意味着在总共50朵Setosa鸢尾花中，有41朵的`petal_length ≤ 1.6`；
* 根据CDF也可以得到所有Setosa鸢尾花的`petal_length ≤ 1.9`。

### 一张图中查看三种类型鸢尾花的单变量CDF


```python
# Plots of CDF of petal_length for various types of flowers.

# Misclassification error if you use petal_length only.

# setosa
counts, bin_edges = np.histogram(iris_setosa['petal_length'], bins=10, density=True)
pdf = counts/(sum(counts))
print('setosa_pdf:', pdf)
print('setosa_bin_edges:',bin_edges)
cdf = np.cumsum(pdf)
plt.plot(bin_edges[1:], pdf, label='setosa_pdf')
plt.plot(bin_edges[1:], cdf, label='setosa_cdf')

# virginica
counts, bin_edges = np.histogram(iris_virginica['petal_length'], bins=10, density=True)
pdf = counts/(sum(counts))
print('virginica_pdf:',pdf)
print('virginica_bin_edges:',bin_edges)
cdf = np.cumsum(pdf)
plt.plot(bin_edges[1:], pdf, label='virginica_pdf')
plt.plot(bin_edges[1:], cdf, label='virginica_cdf')

# versicolor
counts, bin_edges = np.histogram(iris_versicolor['petal_length'], bins=10, density=True)
pdf = counts/(sum(counts))
print('versicolor_pdf:',pdf)
print('versicolor_bin_edges:',bin_edges)
cdf = np.cumsum(pdf)
plt.plot(bin_edges[1:], pdf, label='versicolor_pdf')
plt.plot(bin_edges[1:], cdf, label='versicolor_cdf')

plt.legend()
plt.show()
```

    setosa_pdf: [0.02 0.02 0.04 0.14 0.24 0.28 0.14 0.08 0.   0.04]
    setosa_bin_edges: [1.   1.09 1.18 1.27 1.36 1.45 1.54 1.63 1.72 1.81 1.9 ]
    virginica_pdf: [0.02 0.1  0.24 0.08 0.18 0.16 0.1  0.04 0.02 0.06]
    virginica_bin_edges: [4.5  4.74 4.98 5.22 5.46 5.7  5.94 6.18 6.42 6.66 6.9 ]
    versicolor_pdf: [0.02 0.04 0.06 0.04 0.16 0.14 0.12 0.2  0.14 0.08]
    versicolor_bin_edges: [3.   3.21 3.42 3.63 3.84 4.05 4.26 4.47 4.68 4.89 5.1 ]



![png](/assets/simple-eda/output_32_1.png)


**通过可视化，可以得到如下的结论：**

* 如果 `petal_length ≤ 2`，则鸢尾花的类型为Setosa，并且正确率接近于100%；
* 如果 `petal_length ＞ 2`并且 `petal_length ≤ 5`：
    * **鸢尾花种类为Virginica。** 此结论的正确性可能只有10%，因为在`petal_length = 5`情况下，CDF的值为10，同理，再次区间判定结果有90%的错误可能；
    * **鸢尾花种类为Versicolor。** 此结论的正确率为95%，因为在`petal_length = 5`时，Virginica的CDF值为95。
* 当 `petal_length`位于5到7之间，并且如果在此处将一个鸢尾花的种类定为Virginica，则正确预测该种类的可能性为90%，10%的可能性为Versicolor。

## 箱须图（Box-and-Whisker Plots）

箱形图（或箱须图）以有助于变量之间比较的方式显示定量数据的分布 Box显示数据集的四分位数，而Whisker显示其余分布。

箱须图是显示数据分布的一种标准化方法，该方法基于以下五个数据的摘要绘制：

* 最小值
* 最大值
* 中位数
* 第一个四分位数
* 第三个四分位数

在一个简单的箱形图中，中心矩形跨越第一个四分位数到第三个四分位数（四分位数间距或IQR）。

**Box Plot**


```python
# Box-plot with whiskers: another method of visualizing the 1-D scatter plot more untuitivey.

# NOTE: In the plot below, a technique call inter-quartile range is used in plotting the whiskers.
# Whiskers in the plot below donot correposnd to the min and max values.

# Box-plot can be visualized as a PDF on the side-ways.

sns.boxplot(x='species', y='petal_length', data=iris)
plt.show()
```


![png](/assets/simple-eda/output_35_0.png)


**Whisker Plot**


```python
# A violin Plot combines the benefits of the previous two plots and simplifies them

# Denser regions of the data are fatter, and sparser ones thinner in a violin plot

sns.violinplot(x='species', y='petal_length', data=iris, size=8)
plt.show()
```


![png](/assets/simple-eda/output_37_0.png)


## 仓促的结语

在本文中，粗略的对数据科学问题的前期工作---探索性数据分析，进行了简单的介绍，从中可以了解到如何进行数据的EDA，并从EDA中了解到数据的深层特性，对后续的特征抽取和建模具有非常大的意义。

> 在文中，部分内容并没有深入进行介绍，与其说本文是介绍EDA，倒不如是针对EDA阶段如何一步一步的深入到数据内部的简单了解，希望对您有帮助。
