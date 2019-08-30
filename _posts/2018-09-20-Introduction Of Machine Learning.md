---
layout: post
author: Robin
title: 机器学习基础介绍
tags: 机器学习
categories:
- 机器学习
--- 

**机器学习是一门从数据中提取知识的技术。** 它是统计学、人工智能和计算机科学的交叉研究领域，被常被称为**预测分析**、**统计学习**。机器学习方法的应用近年来在日常生活中无处不在。从自动推荐看哪部电影、点什么食物或买什么东西，到个性化的在线收音机、智能化在线教育，再到从照片中找到你的朋友等等需要现代网站和设备的核心都是机器学习算法。当你查看例如Facebook、Amazon、Netflix、Weibo、Twitter等复杂网站时，很可能网站的每个部分都包含了多个机器学习模型。

除了商业应用之外，机器学习已经对数据驱动的研究方式产生了巨大的影响。机器学习相关的技术、工具已经应用于各种科学问题，例如理解恒星、发现遥远的行星、发现新的粒子、分析DNA序列以及提供个性化的癌症治疗等。

但是，为了从机器学习中获益，你的应用程序可能并不需要大规模。在本部分，将解释为什么机器学习变的如此的流行，并讨论使用机器学习可以解决哪些问题。然后，将展示如何构建你的第一个机器学习模型等。

## 为什么是机器学习？

较早期的“智能”应用程序，许多的系统使用“if”和“else”硬编码规则来处理数据或者根据用户的输入进行调整。想象一个垃圾邮件过滤器，其工作是适当的移动电子邮件到垃圾邮件文件夹。你可以构造一个垃圾邮件词库黑名单，并返回垃圾邮件标记 *1*。这是一个使用专家设计的规则来实现“智能”应用程序的例子。手工创建决策规则对于一些应用程序是可行的，特别是那些人类对建模过程有很好理解的应用程序，然而，使用手工编码的规则进行决策有两点主要缺点：

* 做出决策所需的规则逻辑是针对特定的单个域和任务的，一些业务改变，整个系统可能都需要重写；
* 设计规则需要深刻理解人类专家应该如何做出决定。

这种编码方法失败的一个典型例子是检测图像中的人脸。如今，每个智能手机都能在图像中检测出人脸，然而，这项技术直到2001年，面部检测才有了进展。这项技术之前面临的主要问题是，像素被计算机“感知”的方式与人类感知面部的方式非常不同，这种表现上的差异使得人类基本不可能想出一套好的规则来描述数字图像中人脸的构成。

然而，使用机器学习，仅仅呈现具有大量面部图像的程序就足以让算法确定识别面部需要什么特征。

## 机器学习能解决的问题

最成功的机器学习算法是那些通过从已知例子中归纳出来使决策过程自动化的算法。在称为**监督学习**的设定中，用户向算法提供一对输入和期望的输出，算法会自动找到一种方法，在给定输入的情况下产生期望的输出。特别地，这种算法能够在没有人帮助的情况下为它以前从未遇见的输入创建输出。例如上述垃圾邮件分类的示例，使用机器学习，用户向算法提供大量历史电子邮件（输入），以及关于这些电子邮件中是否有垃圾邮件的信息（期望的输出），算法变回找到一种分别是否为垃圾邮件的方法，当给定一个新的电子邮件，该算法将产生一个关于新邮件是否是垃圾邮件的预测输出。

从输入/输出对中学习的机器学习算法被称为**监督学习**算法，因为“教师”以他们学习的每个示例的期望输出向算法提供监督。虽然创建输入和输出的数据集常常是费时费力的手工过程，但是对于受监督的学习算法来说，却是易于理解的，并且它们的性能易于测量。如果你的应用程序可以被描述为受监督的学习问题，并且你能够创建包含所需结果的数据集，则机器学习可能能够解决你的问题。

有监督的机器学习任务示例包括但不限于如下几种：

* *从信封上手写的数字中识别邮政编码*

这里的输入是手写数字的扫描，所需的输出时邮政编码中的实际数字。要创建用于构建机器学习模型的数据集，需要收集需要的信封。然后可以自己读取邮政编码，并将数字存储为你想要的结果。

* *基于医学图像判断肿瘤是否良性*

这的输入是图像，输出时肿瘤是否良性。要创建用于构建模型的数据集，需要一个医学图像数据库，还需要专家的意见，所以需要查看所有的图像，并决定哪些肿瘤是良性的，哪些不是，甚至可能需要做出超出图像内容的附加诊断，以确定图像中的肿瘤是否是癌性的。

* *信用卡交易中欺诈行为的侦测*

这里的输入是信用卡交易的记录，输出是该交易是否可能欺诈。假设你是分发信用卡的金融机构，收集数据意味着需要存储所有的交易事务，并且如果出现欺诈的事务，需要进行记录。

以上示例需要注意的一件有趣的事情是，尽管输入和输出看起来都非常简单，易于理解，但是这三个任务的数据收集过程却大不相同。虽然阅读信封是费力的，但是却是容易和廉价的；另一方面，获得医学成像和诊断不仅需要昂贵的机器，而且需要稀有和昂贵的专家知识，更不用说伦理问题和隐私问题了；在检测信用卡欺诈的例子中，数据收集简单的多，你的客户会提供给你想要的输出，因为他们会报告欺诈行为，你必须要做的是获得欺诈性和非欺诈性的输入/输出对。

**无监督算法** 是另一种机器学习算法。在无监督学习中，只有输入数据是已知的，并且没有已知的输出数据被赋予算法。虽然这些方法有很多成功的应用，但他们通常难以理解和评估。

无监督学习的例子包括：

* *在一组博客文章中识别主题*

如果你用大量的文本数据集合，你可能需要总结它们并找到其中的流行主题，你可能实现并不知道这些主题是什么，或者可能有多少主题，因此，没有已知的输出。

* *将客户细分为具有相似偏好的群体*

给定一组客户记录，你可能希望识别哪些客户是相似的，以及是否具有相似的偏好。例如一个购物网站，客户可能是“父母”、“书虫”、“玩家”等，因为你事先不知道这些群体可能是什么，甚至不知道有多少群体，所以你没有已知的输出。

* *检测网站的异常访问模式*

为了识别滥用和错误，找到与规范不同的访问模式通常对于网站来说是有益的。每个异常模式可能是非常不同的，并且你可能没有任何异常行为的记录实例，因为在这个例子中，你只能观察流量，并且你不知道什么构成了正常和异常的行为，所以这是一个无监督的问题。

对于有监督和无监督的学习任务，计算机能够理解的输入数据的表示是至关重要的，通常把数据看做表格是有帮助的。你想要推理的每个数据点（每个电子邮件、每个客户、每个事务）都是一行，描述该数据点（例如，客户的年龄或者事务的数量或者位置）的每个属性都是一列，你可以根据用户的年龄、性别、创建账号时间以及从网上购买的频率来描述用户，你可以通过每个像素的灰度值来描述肿瘤的图像，或者可以使用肿瘤的大小、形状和颜色等。

这里的每个实体或者行都称之为机器学习中的示例或数据点，而列（描述这些实体的属性）称之为特征。

## 了解你的任务，认识你的数据

在机器学习过程中最重要的部分是理解你正在处理的数据以及这些数据与你想要解决的任务之间的关系。随机选择一个算法并把数据丢给算法通常是无效的，在开始构建模型之前，必须了解数据集中发生的事情。每个算法在什么样的数据和什么样的问题设置下的表现是不同的，当你正在构建一个机器学习解决方案时，首先你必须回答一下几个问题或者至少要记住一下几个问题：

* 我想要解决什么样的问题？所收集的数据能回答这个问题吗？
* 将我的问题作为机器学习问题的最佳方式是什么？
* 我收集了足够的数据来表示我想解决的问题吗？
* 我提取了数据的哪些特征，这些特征会是正确的预测成为可能吗？
* 我如何衡量机器学习在我的应用程序中是成功的？
* 机器学习解决方案将如何与我的研究或商业产品的其他部分交互？

在较大的场景中，机器学习中的算法和方法只是解决特定问题的的一小部分，并且始终要牢记全局。很多人花费大量的时间构建复杂的机器学习解决方案，结果却发现他们没有解决正确的问题。

当深入研究机器学习的技术时，很容易忽略最终的目标，虽然我们在这里不会详述这些问题，但是仍然鼓励在开始构建机器学习模型时，记住可能正在显示或者隐式做出的所有假设。

## 为什么选择Python？

Python语言已经成为了一种通用语言，能够处理很多应用的科学数据。它结合了通用编程语言的优势和易于在特定领域使用的特定脚本语言，如MATLAB或R。Python具有用于数据加载、可视化、统计、自然语言处理、图像处理等库，这个庞大的工具库为数据科学提供了大量的通用和专用的功能，使用python的主要优点之一是能够直接与代码进行交互，使用终端或者其他工具，例如**Jupyter Notebook**。机器学习和数据分析都是不断的迭代过程，也被称为数据驱动分析，因此对于这样的过程来说，有必要有一些快速迭代和易于交互的工具。

作为一种通用编程语言，Python还允许创建复杂的用户图形界面（GUI）和Web服务，以及集成到现有的系统中等。

## scikit-learn

*scikit-learn* 是一个开放的源码项目，这意味着它可以自由使用和开发，任何人都可以轻松获得其源代码，以及了解内部的核心。*scikit-learn* 项目在不断地被开发和改进，并且它有一个非常活跃的用户社区，包含许多先进的机器学习算法，以及关于每个算法的[综合文档](http://scikit-learn.org/stable/documentation)。*scikit-learn* 是一个非常流行的工具，也是最著名的机器学习Python库，被广泛应用于工业和学术界，网路上有大量的教程和代码片段，*scikit-learn* 可以与其他Python工具一起工作。

在阅读本书时，建议你浏览*scikit-learn* 的[用户指南](http://scikit-learn.org/stable/user_guide.html)和API文档，了解关于每个算法的更多细节和更多选项。在线文档非常全面，本书将为你提供机器学习中的所有先决条件，以详细了解它。

## 安装 scikit-learn

*scikit-learn* 依赖两个其他的Python库， *NumPy* 和 *SciPy*。为了绘图和交互开发，你还应该安装 *matplotlib*、*IPython* 和 *Jupyter Notebook*。这里建议你使用以下某一种Python环境管理工具：

**[Anaconda](https://store.continuum.io/cshop/anaconda/)**

Python能够用于大规模数据处理、预测分析和科学计算，*Anaconda* 可与 *NumPy*、*SciPy*、*matplotlib*、*Pandas*、*IPython* 、*Jupyter Notebook*、*scikit-learn*一起工作，可以在Mac OS、Windows和Linux上安装，它是一个非常方便的集成解决方案，是我们没有进行科学安装Python经验的人的建议解决方案。

**[Enthought Canopy](https://www.enthought.com/product/canopy/)**

另一种用于科学计算的集成Python管理工具，可与*NumPy*、*SciPy*、*matplotlib*、*Pandas*、*IPython* 一起工作，但是其免费版本不能与*scikit-learn*一起工作，如果需要使用*scikit-learn*则需要付费或者申请学术许可证，便可获得其免费访问付费订阅版本的Enthought Canopy，Enthought Canopy目前仅支持Python2.7.x，能够工作在Mac OS、Windows、Linux上。

**[Python(x,y)](http://python-xy.github.io/)**

一个免费的Python管理工具，用于科学计算，特别是Windows上。*Python（X，Y）* 可与  *NumPy*、*SciPy*、*matplotlib*、*Pandas*、*IPython* 、*Jupyter Notebook*、*scikit-learn*一起工作。

如果你已经安装好了Python，也可以使用**pip**安装所有需要的包：

```python
  pip install numpy scipy matplotlib ipython scikit-learn pandas
```

## 基本库和工具

了解了什么是 *scikit-learn* 以及如何使用它很重要，但是还有一些其他的类库可以提高使用体验。*scikit-learn* 是建立在 *NumPy* 和 *SciPy* 之上的，除了*NumPy* 和 *SciPy* ，我们还将使用 *Pandas*、*matplotlib* 以及 *Jupyter Notebook*，*Jupyter Notebook* 是一个基于浏览器的交互式编程环境，简言之，以下这些工具是应该了解的内容，以便能够在*scikit-learn* 的使用中获得更多的益处。

### Jupyter Notebook

*Jupyter Notebook* 是一个基于浏览器的交互式编程环境。它是进行探索性数据分析的一个很好的工具，被数据科学家广泛的使用。虽然 *Jupyter Notebook* 支持需要的编程语言，但是我们只需要Python支持即可。*Jupyter Notebook* 使得合并代码、文本和图像变的很容易，而且这本书实际上是在 *Jupyter Notebook* 中编写的。

### NumPy

*NumPy* 是Python中科学计算的基本包之一，它包含多维数组的功能、高级数学函数，如线性代数和傅里叶变换，以及随机数发生器等等。

在 *scikit-learn* 中，*NumPy* 数组是基本的额数据结构，*scikit-learn* 以 *NumPy* 数组的形式获取数据，你所使用的任何数据都必须转换为 *NumPy* 数组。*NumPy* 的核心功能是 **ndarray** 类，一个多维数组结构，数组中的所有元素必须是相同类型的，一个*NumPy*数组看起来是如下样子的：


```python
import numpy as np

x = np.array([[1, 2, 3], [4, 5, 6]])
print("x:\n{}".format(x))
```

    x:
    [[1 2 3]
     [4 5 6]]


在本书中，将多次使用*NumPy*，我们将把*NumPy* 的 **ndarray** 类的对象称为 “NumPy 数组” 或仅仅是 “数组”。

### SciPy

*SciPy* 是Python中的科学计算功能的集合，它提供了高级线性代数程序、数学函数优化、信号处理、特殊数学函数和统计分析功能等，*scikit-learn* 从 *SciPy* 的功能集合中提取用于实现其算法的功能。*SciPy* 对于我们来说最重要的部分是 **scipy.sparse**，它提供了稀疏矩阵的运算，这是*scikit-learn*中用于数据的另一种表示，每当我们想要存储一个包含零点的二维数组时，便于使用稀疏矩阵，如下：


```python
from scipy import sparse

# 创建一个具有对角线的2D NumPy数组，并且在其他任何地方创建零点
eye = np.eye(4)
print("NumPy array: \n{}".format(eye))
```

    NumPy array: 
    [[1. 0. 0. 0.]
     [0. 1. 0. 0.]
     [0. 0. 1. 0.]
     [0. 0. 0. 1.]]



```python
# 将NUMPY数组转换成CSR格式中的一个SISPY稀疏矩阵。
# 仅存储非零项

sparse_matrix = sparse.csr_matrix(eye)
print("\nSciPy sparse CSR matrix:\n{}".format(sparse_matrix))
```

    
    SciPy sparse CSR matrix:
      (0, 0)	1.0
      (1, 1)	1.0
      (2, 2)	1.0
      (3, 3)	1.0


通常不可能创建稀疏数据的密集表示（因为它们不适合于内存），所以我们需要直接创建稀疏表示。下面是使用COO格式创建与以前一样的稀疏矩阵的方法：


```python
data = np.ones(4)
print("data: \n{}".format(data))
row_indices = np.arange(4)
print("row_indices: \n{}".format(row_indices))
col_indices = np.arange(4)
print("col_indices: \n{}".format(col_indices))
eye_coo = sparse.coo_matrix((data, (row_indices, col_indices)))
print("eye_coo: \n{}".format(eye_coo))
```

    data: 
    [1. 1. 1. 1.]
    row_indices: 
    [0 1 2 3]
    col_indices: 
    [0 1 2 3]
    eye_coo: 
      (0, 0)	1.0
      (1, 1)	1.0
      (2, 2)	1.0
      (3, 3)	1.0


*SciPy* 稀疏矩阵的更多细节可以在[SciPy讲义](http://www.scipy-lectures.org/)中找到。

### matplotlib

*matplotlib* 是Python的主要科学绘图库，它提供了用于制作高质量可视化的功能，如线形图、直方图、散点图等。可视化数据和分析的不同在于可视化可以给你提供更加直观的数据理解。我们使用*matplotlib* 进行所有的可视化，在 *Jupyter Notebook* 中工作时，可以使用 *%matplotlib notebook* 和 *%matplotlib inline* 在浏览器中直接使用可视化。


```python
%matplotlib inline
import matplotlib.pyplot as plt

# Generate a sequence of numbers from -10 to 10 with 100 steps in between
x = np.linspace(-10, 10, 100)
# Create a second array using sine
y = np.sin(x)
# The plot function makes a line chart of one array against another
plt.plot(x, y, marker = 'x')
```




    [<matplotlib.lines.Line2D at 0x111be57b8>]




![png](/assets/Introduction_Of_Machine_Learning/output_18_1.png)


### pandas

*pandas* 是一个用于数据处理和分析的Python库，它围绕了一个叫做**DataFrame**的数据结构进行构建。简单的说，*pandas* 数据文件是一个表，类似于Excel电子表格，*pandas* 提供了大量的方法来修改和操作这个表，特别是，*pandas* 允许类似SQL的查询和连接。与 *NumPy* 不同，要求数组中的所有条目都具有相同的类型，*pandas* 允许每个列具有单独的类型（例如整数、日期、浮点数和字符串）。*pandas* 提供的另一个有价值的工具是它能够从各种文件格式和数据库中获取信息，如SQL、Excel文件和逗号分隔值（CSV）文件等。详细介绍*pandas* 的功能不再本书的范围之内，然而Wes McKinney (O’Reilly, 2012)的[Python数据分析](http://shop.oreilly.com/product/0636920023784.do)提供了一个很好的指南。下面是使用字典数据创建数据文件的一个例子：


```python
import pandas as pd

# create a simple dataset of people
data = {'Name':['John', 'Anna', 'Peter', 'Linda'],
       'Location':['New York', 'Paris', 'Berlin', 'London'],
       'Age':[24, 14, 53, 33]}

data_pandas = pd.DataFrame(data)
# IPython.display allows "pretty printing" of dataframes 
# in the Jupyter notebook
display(data_pandas)
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
      <th>Name</th>
      <th>Location</th>
      <th>Age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>John</td>
      <td>New York</td>
      <td>24</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anna</td>
      <td>Paris</td>
      <td>14</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Peter</td>
      <td>Berlin</td>
      <td>53</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Linda</td>
      <td>London</td>
      <td>33</td>
    </tr>
  </tbody>
</table>
</div>


查询此表有几种可能的方式。例如：


```python
# Select all rows that have an age column greater than 30
display(data_pandas[data_pandas.Age > 30])
# Select all rows that have an age column equal 53
display(data_pandas[data_pandas['Age'] == 53])
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
      <th>Name</th>
      <th>Location</th>
      <th>Age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Peter</td>
      <td>Berlin</td>
      <td>53</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Linda</td>
      <td>London</td>
      <td>33</td>
    </tr>
  </tbody>
</table>
</div>



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
      <th>Name</th>
      <th>Location</th>
      <th>Age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Peter</td>
      <td>Berlin</td>
      <td>53</td>
    </tr>
  </tbody>
</table>
</div>


## 本书所用类库的版本

本书使用了如下一些类库，各个类库的版本信息如下：


```python
import sys
print('Python version: {}'.format(sys.version))

import pandas as pd
print('pandas version: {}'.format(pd.__version__))

import matplotlib
print('matplotlib version: {}'.format(matplotlib.__version__))

import numpy as np
print('numpy version: {}'.format(np.__version__))

import scipy as sp
print('scipy version: {}'.format(sp.__version__))

import IPython
print('IPython version: {}'.format(IPython.__version__))

import sklearn
print('sklearn version: {}'.format(sklearn.__version__))
```

    Python version: 3.6.6 |Anaconda, Inc.| (default, Jun 28 2018, 11:07:29) 
    [GCC 4.2.1 Compatible Clang 4.0.1 (tags/RELEASE_401/final)]
    pandas version: 0.23.3
    matplotlib version: 2.2.2
    numpy version: 1.14.5
    scipy version: 1.1.0
    IPython version: 6.4.0
    sklearn version: 0.19.1


虽然使用精确匹配的版本并不重要，但是需要注意的是尽量保持你的*scikit-learn*版本最新。

既然我们已经设置好了开发环境，让我们深入研究及其学习的第一个应用吧。

## 第一个机器学习应用：鸢尾花分类

在本部分，我们将实现一个简单的机器学习应用以及构建机器学习模型。在这个过程中，我们将介绍一些核心的概念和术语等。

假设一个植物爱好者有兴趣区分他所发现的一些鸢尾花的种类，他收集了一些与每个鸢尾花相关的测量数据：花瓣的长度和宽度、萼片的长度和宽度，均已厘米为单位。

![](/assets/Introduction_Of_Machine_Learning//iris_petal_sepal.png)

收集到的数据经过植物专家的鉴定属于刚毛鸢尾、云彩鸢尾或维珍鸢尾。对于这些测量，他可以确定每个鸢尾属属于哪种。

我们的目标是建立一个机器学习模型，可以这些鸢尾花已知物种的测量数据，最终能够预测新的鸢尾花的物种。

因为我们有测量了的鸢尾花所属的种类，这是一个有监督的学习问题。在这个问题中，我们要预测几种选择中的一个（具体所属的种类），这是一个分类问题的例子，可能的输出（不同种类的鸢尾）被称为*类*，由于数据集中每个鸢尾都可能属于三个种类中的一个，因此这是一个三分类的问题。

单个数据点的期望输出时这种花的种类，对于一个特定的数据点，它所属的五种种类被称为它的*标签*。

### 遇见数据

在这个例子中我们将使用iris数据集，机器学习和数据统计中经典的数据集，该数据集包含在了*scikit-learn*中，我们可以直接使用 *load_iris* 函数加载它：


```python
from sklearn.datasets import load_iris
iris_dataset = load_iris()
```

由函数 *load_iris* 加载的数据集返回的iris对象是一个堆对象，比较类似于字典。它包含的键和值如下：


```python
print("Keys of iris_dataset: \n{}".format(iris_dataset.keys()))
```

    Keys of iris_dataset: 
    dict_keys(['data', 'target', 'target_names', 'DESCR', 'feature_names'])


其中 *DESCR* 字段是对数据集的简要描述，我们可以简要的查看一下其描述内容（这里仅仅查看部分内容，你可以查看全部内容）：


```python
print(iris_dataset['DESCR'][:470] + '\n...')
```

    Iris Plants Database
    ====================
    
    Notes
    -----
    Data Set Characteristics:
        :Number of Instances: 150 (50 in each of three classes)
        :Number of Attributes: 4 numeric, predictive attributes and the class
        :Attribute Information:
            - sepal length in cm
            - sepal width in cm
            - petal length in cm
            - petal width in cm
            - class:
                    - Iris-Setosa
                    - Iris-Versicolour
                    - Iris-Virginic
    ...


*target_names* 的值是一个字符串数组，包含了我们想要预测的花的种类：


```python
print("Target names: {}".format(iris_dataset['target_names']))
```

    Target names: ['setosa' 'versicolor' 'virginica']


*feature_names* 是一个字符串列表，给出了每个特征的描述：


```python
print("Feature names: \n{}".format(iris_dataset['feature_names']))
```

    Feature names: 
    ['sepal length (cm)', 'sepal width (cm)', 'petal length (cm)', 'petal width (cm)']


数据集本身包含了目标和数据，数据包含了萼片长度、萼片宽度、花瓣长度、花瓣宽度的测量数字，保存在一个 *NumPy* 数组中：


```python
print('Type of data: {}'.format(type(iris_dataset['data'])))
```

    Type of data: <class 'numpy.ndarray'>


数据阵列中的行对应着花的种类，而列表示针对每个花采取的四个度量：


```python
print('Shape of data: {}'.format(iris_dataset['data'].shape))
```

    Shape of data: (150, 4)


我们看到阵列包含150种不同花的测量，在机器学习中也被称为**样本**，他们的属性称为**特征**。数据阵列的形状是样本的数量乘以特征的数量，这是*Scikit-learn*中的一个约定，你的数据将始终假定为该形状。以下是前5个样本的特征值：


```python
print('First five columns of data: \n{}'.format(iris_dataset['data'][:5]))
```

    First five columns of data: 
    [[5.1 3.5 1.4 0.2]
     [4.9 3.  1.4 0.2]
     [4.7 3.2 1.3 0.2]
     [4.6 3.1 1.5 0.2]
     [5.  3.6 1.4 0.2]]


从这些数据中，我们可以看出，所有前五朵花的花瓣宽度都是0.2厘米，而第一朵花的萼片最长，为5.1厘米。

目标数据阵列中包含被测量的每一朵花所属的种类，也是一个*NumPy*数组：


```python
print('Type of target: {}'.format(type(iris_dataset['target'])))
```

    Type of target: <class 'numpy.ndarray'>


目标（target）是一个关于每一朵花所属种类的一维的数组：


```python
print('Shape of target: {}'.format(iris_dataset['target'].shape))
```

    Shape of target: (150,)


该目标数据被编码为从0到2的整数：


```python
print('Target:\n{}'.format(iris_dataset['target']))
```

    Target:
    [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
     1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 2
     2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2
     2 2]


该目标值与*target_names*值所对应：0表示setosa、1表示versicolor、2表示virginica。

### 衡量成功：训练集与测试集

我们希望建立一个机器学习模型，使用这个数据集，可以预测鸢尾花的种类。但是在将模型应用在新的预测之前，我们需要知道模型是否真的有效，也就是说，我们是否应该相信模型的预测。

不好的是，我们不能直接使用我们的数据集来构建模型并评估模型，因为我们的模型总是可以简单的记住整个数据集，并且总是会正确的预测训练数据集中任何数据点的正确标签。这种“记忆”并不表明我们的模型能够很好的泛化（也就是说，模型是否也会在新数据上同样表现良好。）。

为了评估模型的性能，我们展示了我们有标签的新数据（以前没有见过的数据），通常情况下会分裂我们收集的标记数据（在这里，就是150个花的测量）分为两个部分，一部分用于构建我们的机器学习模型，被称为训练数据或者训练集，其余的数据将用来评估模型的工作如何，被称为测试数据或测试集。

*scikit-learn* 中包含一个函数，它可以将数据集拆分为训练集和测试集：*train_test_split* 函数。该函数会将数据集中的75%行作为训练集，并连同这些数据所对应的标签一起提取，剩下的25%的数据，连同剩下的标签，被声明为测试集。当然你也可以自定义训练集和测试集所占比例，但是25%的测试集比例是一个很好的经验法则。

在*scikit-learn* 中，数据通常使用大写字母*X*表示，而标签测用小写字母*y*表示。这其实是受到标准公式 *f(x) = y* 的启发，其中*x*是函数的输入，*y*是函数的输出，遵循数学中的更多约定，我们使用大写字母*X*，因为数据是二维数组（矩阵），小写字母*y*是因为目标是一维数组（向量）。

让我们在数据上调用 *train_test_split*，并使用上述命名规则：


```python
from sklearn.model_selection import train_test_split

X = iris_dataset['data']
y = iris_dataset['target']
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=0)
```

在实际进行数据集拆分之前，*train_test_split* 函数内部会使用伪随机数发生器对数据进行洗牌操作。如果我们仅仅只是使用数据集中的后25%作为测试集，那么所有的数据将只具有标签2，因为数据点是按照标签来排序的。使用只包含三个类之一的测试集，并不能评估出我们的模型泛化能力，因此对数据进行洗牌操作，以确保测试数据集包含来自所有类的数据。

为了确保如果多次运行相同的函数，我们将获得相同的输出，这里使用*random_state*参数为伪随机数发生器提供一个固定的种子，这会使得输出确定。

*train_test_split*函数的输出是 *X_train, X_test, y_train, y_test*，均为*NumPy*数组，*X_train*包含数据集的75%行，*X_test*包含剩下的25%行：


```python
print('X_train shape: {}'.format(X_train.shape))
print("y_train shape: {}".format(y_train.shape))
```

    X_train shape: (112, 4)
    y_train shape: (112,)



```python
print("X_test shape: {}".format(X_test.shape)) 
print("y_test shape: {}".format(y_test.shape))
```

    X_test shape: (38, 4)
    y_test shape: (38,)


## 第一件事：看看你的数据

在建立机器学习模型之前通常的一个好的思路是检查数据，看看所面临的问题是否不使用机器学习可以解决，或者查看期望的输出信息是否可能不包含在数据中等。

除此之外，检查数据是发现数据异常和数据某些特殊性等的一个很好的方法，也许在你的鸢尾花数据集中有一些数据是以英寸为单位而不是厘米为单位的。在现实世界中，数据的不一致和意外的测量都是非常普遍的。

检查数据最好的方式之一就是可视化数据，其中一种可视化方法就是使用散点图。数据的散点图沿着x轴放置一个特征，另一个沿y轴放置一个特征，并为每个数据点画一个点，但是不好的是，计算机屏幕只有两个维度，这使得我们一次只能绘制两个或三个特征，如果要绘制超过三个特征的数据集是困难的。解决这个问题的一个方法是绘制一对图，它查看所有可能的特征对，如果你有一小部分特征，比如我们这里的四个，这是相当合理的。但是，您应该记住，配对图不会同时显示所有特性的交互，因此以这种方式可视化数据时，可能不会显示数据的一些有趣方面。

下图是数据集中特征的对图，数据点根据鸢尾花所属的种类进行着色，为了创建该图，首先将*NumPy*数据转换为*Pandas*数据文件，*Pandas*有一个功能，创建配对图称为散射矩阵。该矩阵的对角线填充有每个特征的直方图：


```python
import mglearn
import seaborn as sns
sns.set()

# create dataframe from data in X_train
# label the columns using the strings in iris_dataset.feature_names
iris_dataframe = pd.DataFrame(X_train, columns=iris_dataset.feature_names)
# create a scatter matrix from the dataframe, color by y_train 
pd.plotting.scatter_matrix(iris_dataframe, c=y_train, figsize=(15, 15), marker='o',
                           hist_kwds={'bins': 20}, s=60, alpha=.8, cmap=mglearn.cm3)
```




    array([[<matplotlib.axes._subplots.AxesSubplot object at 0x1c160d2550>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c160f8e80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c16127940>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c16153400>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1c1617ae80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c1617aeb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c161d5400>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c161fce80>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1c16229940>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c16258400>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c1627ee80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c162ad940>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1c162dc400>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c16301e80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c1632f940>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1c16361400>]],
          dtype=object)




![png](/assets/Introduction_Of_Machine_Learning/output_56_1.png)


**对角线部分：** 每个特征的直方图。体现每个特征的密度分布情况。

**非对角线部分：** 两个变量之间分布的关联散点图。将任意两个变量进行配对，以其中一个为横坐标，另一个为纵坐标，将所有的数据点绘制在图上，用来衡量两个变量的关联度（Correlation）。

从图中我们可以看到，通过萼片和花瓣的测量数据似乎可以很好的区分鸢尾花的种类，这意味着机器学习模型能够学会如何分类它们。

为了能够更加直观的解释为什么通过该图可以确定机器学习模型能够分类它们，让我们通过创建一个散布矩阵中花瓣的长度（Petal Length）和花瓣宽度（Petal Width）的散点图来展开这一扩展。


```python
df = pd.DataFrame(iris_dataset.data, columns=iris_dataset.feature_names) 
df['Target'] = pd.DataFrame(iris_dataset.target) 


plt.clf()
plt.figure(figsize = (10, 6))
names = iris_dataset.target_names
colors = ['b','r','g']
label = (iris_dataset.target).astype(np.int)
plt.title('Petal Width vs Petal Length')
plt.xlabel(iris_dataset.feature_names[2])
plt.ylabel(iris_dataset.feature_names[3])
for i in range(len(names)):
 bucket = df[df['Target'] == i]
 bucket = bucket.iloc[:,[2,3]].values
 plt.scatter(bucket[:, 0], bucket[:, 1], label=names[i]) 
plt.legend()
plt.show()
```


    <Figure size 432x288 with 0 Axes>



![png](/assets/Introduction_Of_Machine_Learning/output_58_1.png)


乍一看，我们可以看到蓝色的点(Setosa类)可以很容易地通过画一条线来分隔，并将其与其他类隔离开来。但是其他两个类呢?

让我们检查另一种更确定的方法。

**计算几何学**

在这种方法中，我们将使用凸包（Convex Hull）来检查一个特定的类是否是线性可分的。简而言之，凸包代表了一组数据点(类)的外边界，这就是为什么有时它被称为凸包。

当测试线性可分性时使用凸包的逻辑是相当直接的，可以这样说:

> 如果X和Y的凸包的交点是空的，那么两个类X和Y是线性可分的。

一种快速的方法来查看它是如何工作的，就是将每个类的凸包的数据点可视化。我们将绘制凸包边界，以直观地检查交点。我们将使用Scipy库来帮助我们计算凸包。更多信息请参阅下方Scipy文档地址。

* 地址：[https://docs.scipy.org/doc/scipy/reference/generated/scipy.spatial.ConvexHull.html](https://docs.scipy.org/doc/scipy/reference/generated/scipy.spatial.ConvexHull.html)


```python
from scipy.spatial import ConvexHull
 
plt.clf()
plt.figure(figsize = (10, 6))
names = iris_dataset.target_names
label = (iris_dataset.target).astype(np.int)
colors = ['b','r','g']
plt.title('Petal Width vs Petal Length')
plt.xlabel(iris_dataset.feature_names[2])
plt.ylabel(iris_dataset.feature_names[3])
for i in range(len(names)):
 bucket = df[df['Target'] == i]
 bucket = bucket.iloc[:,[2,3]].values
 hull = ConvexHull(bucket)
 plt.scatter(bucket[:, 0], bucket[:, 1], label=names[i]) 
 for j in hull.simplices:
     plt.plot(bucket[j,0], bucket[j,1], colors[i])
plt.legend()
plt.show()
```


    <Figure size 432x288 with 0 Axes>



![png](/assets/Introduction_Of_Machine_Learning/output_60_1.png)


至少从直观上看，Setosa是一个线性可分的类。换句话说，我们可以很容易地画出一条直线，将Setosa类与非Setosa类分开。Versicolor类和Versicolor类都不是线性可分的，因为我们可以看到它们两个之间确实有一个交集。

## 构建你的第一个模型：K-最近邻模型（k-Nearest Neighbors，kNN）

了解了数据之后，我们现在开始构建一个实际的机器学习模型。在*scikit-learn*中有很多的分类算法可以使用，这里我们使用易于理解的 *k-Nearest Neighbors* 分类器。该分类器仅需要存储的训练集，为了能够对新的数据点进行预测，该算法在训练集中找到距离新数据点最接近的点，然后，将这个训练点的标签分配给新的数据点。

*kNN* 中的 *k* 表示在训练中我们可以考虑多少固定数目的数据点（例如最近的3个或者5个），而不是仅仅使用距离新数据点最新的点，然后，我们可以使用这些距离较近的数据点指点的多数进行最终的标签分配，为了简单起见，这里我们仅仅使用一个距离最近的点。

*scikit-learn* 中的所有机器学习模型都在自己的类中实现的，通常被称为 *估计器类*。*kNN* 分类算法是在 *neighbors* 模块的 *KNeighborsClassifier* 类中实现的。在我们实现模型之前，需要将类实例化为一个对象，当我们设置模型参数的额时候，会将 *KNeighborsClassifier* 分类器最终的参数邻居数设置为1（即 *k=1*）：


```python
from sklearn.neighbors import KNeighborsClassifier
knn = KNeighborsClassifier(n_neighbors=1)
```

*knn* 对象封装了用于根据训练数据集建立机器学习模型的算法，以及用于对新数据进行预测的算法，它还保存了算法从训练数据中提取的信息。

为了在训练集上建立模型，我们只需要调用 *knn* 的 *fit* 函数，该函数包含训练数据的 *NumPy* 数组 *X_train* 和对应的训练标签 *NumPy* 数组 *y_train* 数组作为参数：


```python
knn.fit(X_train, y_train)
```




    KNeighborsClassifier(algorithm='auto', leaf_size=30, metric='minkowski',
               metric_params=None, n_jobs=1, n_neighbors=1, p=2,
               weights='uniform')



*fit* 函数返回 *knn* 对象本身（其内部已经根据设定进行修改），因为我们得到了我们的分类器的字符串表示，表明我们在创建模型的时候使用了哪些参数。几乎所有的这些都是默认值，但是你也能够找到 *n_neighbors=1*，这是我们设定的参数。*scikit-learn* 中的大多数模型都需要很多的参数，但是大多数都是速度优化或者用于非常特殊的用例，不必担心在该表示中显示的其他参数。

## 进行预测

有了模型之后，我们就可以使用该模型对新的数据进行预测了。假设我们有一个新的鸢尾花的测量数据：萼片长度5cm、萼片宽度2.9cm、花瓣长度1cm、花瓣宽度0.2cm。该鸢尾花属于哪个类呢？我们可以将这些数据放在 *NumPy* 数组中，并计算数组形状 -- 数组形状应该为样本数 1，特征数 4：


```python
X_new = np.array([[5, 2.9, 1, 0.2]])
print('X_new.shaple: {}'.format(X_new.shape))
```

    X_new.shaple: (1, 4)


你可能注意到了，在构建 *NumPy* 数组的时候，我们使用的是二维的形式，这是因为在 *scikit-learn* 中的数据总是希望具有二维性。

为了进行预测，我们只需要调用 *knn* 对象的 *predict* 函数即可：


```python
prediction = knn.predict(X_new)
print('Prediction: {}'.format(prediction))
print('Predicted target name: {}'.format(iris_dataset.target_names[prediction]))
```

    Prediction: [0]
    Predicted target name: ['setosa']


如果你需要查看属于各个分类的概率，可以调用 *knn* 对象的 *predict_proba* 函数：


```python
prediction_prob = knn.predict_proba(X_new)
print('Prediction prob: {}'.format(prediction_prob))
```

    Prediction prob: [[1. 0. 0.]]


可以看到概率的输出和标签的输出是一致的。

## 评估模型

在之前我们进行了数据集的拆分，分为了训练集和测试集，训练集我们用来训练模型，而测试集的作用便是评估模型了。在测试集中我们已经知道了每个鸢尾花的种类，因此我们可以对测试集数据调用预测函数，并将预测结果和实际的结果进行比较，已确定模型的性能。可以通过计算模型预测的精确度来测量模型的工作情况，精确度是预测正确种类的花朵的比例：


```python
y_pred = knn.predict(X_test)
print('Test set predictions: \n{}'.format(y_pred))
```

    Test set predictions: 
    [2 1 0 2 0 2 0 1 1 1 2 1 1 1 1 0 1 1 0 0 2 1 0 0 2 0 0 1 1 0 2 1 0 2 2 1 0
     2]



```python
print('Test set score: {:.2f}'.format(np.mean(y_pred == y_test)))
```

    Test set score: 0.97


上述方法是计算模型预测的精确度，统计所有预测正确的数目进行去均值计算。你也可以直接调用 *knn* 对象的 *score* 函数进行计算：


```python
print('Test set score: {:.2f}'.format(knn.score(X_test, y_test)))
```

    Test set score: 0.97


对于这个模型，测试集的精确度为 0.97，这意味着我们对测试机中的 97% 的鸢尾花进行了正确的预测，在一些数据假设下，意味着我们可以预期我们的鸢尾花分类模型对新的鸢尾花的识别精确度为 97%。对于一个植物爱好者来说，这种高精度意味着我们的模型具有很高的可信赖度。

## 总结与回归

让我们总结一下我们在这章学到的内容。首先我们介绍了机器学习及其应用，然后讨论了监督学习和无监督学习的区别，并给出了我们将在本书中使用的工具的概述，之后，我们通过对鸢尾花的物理测定，确定了一种预测鸢尾花所属种类的预测任务。我们使用了一个带有正确标签的鸢尾花数据集建立了一个分类模型。

鸢尾花数据集由两个 *NumPy* 数组组成，一个包含数据，在 *scikit-learn* 中称为 *X*，另一个包含正确或期望额输出，称为 *y*，数组 *y* 是一个一维数组，仅包含一个类标签，每个示例的证书范围是0到2。

我们将数据集拆分为训练集和测试集，训练集用于构建我们的模型，测试集用于评估我们的模型。

我们选择了 *k-最近邻* 分类算法，该算法通过考虑训练集中的最近邻来对新的数据点进行预测，该算法在类 *KNeighborsClassifier* 中实现，该类包含构建模型的算法和使用模型进行预测的算法，使用前需要先实例化类并设置参数。然后通过调用拟合函数 *fit* 进行模型的构建，将训练数据 *X_train* 和训练标签 *y_train* 作为参数传递。我们使用评分法对模型进行评估，计算模型的准确性。我们使用测试集和测试标签进行模型的评估，发现我们的模型准确率在97%左右，意味着在测试集上有97%的数据是预测正确的。这给了我们将模型应用到新的数据上的信心，并且相信模型在大约97%的事件内都是正确的。

下面是整个代码实现：


```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.neighbors import KNeighborsClassifier

iris_dataset = load_iris()
X = iris_dataset.data
y = iris_dataset.target

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=0)

knn = KNeighborsClassifier(n_neighbors=1)
knn.fit(X_train, y_train)

print('\n{}'.format(knn))

y_pred = knn.predict(X_test)
print('Test set score: {:.2f}'.format(np.mean(y_pred == y_test)))
```

    
    KNeighborsClassifier(algorithm='auto', leaf_size=30, metric='minkowski',
               metric_params=None, n_jobs=1, n_neighbors=1, p=2,
               weights='uniform')
    Test set score: 0.97

