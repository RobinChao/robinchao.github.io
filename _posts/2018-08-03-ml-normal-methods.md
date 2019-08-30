---
layout: post
author: Robin
title: 机器学习问题的通用方法
tags: 机器学习
categories:
  - 机器学习
--- 

一位数据科学家平均每天处理大量数据，有人说，超过60-70％的时间花在了数据采集、数据清理、数据整理上，使得机器学习模型可以应用于这些数据。本文重点介绍第二部分，即应用机器学习模型，包括预处理步骤。这篇文章中讨论的流水线是我参与过的一百多次机器学习竞赛的结果。必须指出，这里的讨论虽然普通，但非常有用，也存在非常复杂的方法，可供专业人员练习。

我们将在这里使用python！

# 数据

在应用机器学习模型之前，必须将数据转换为表格形式。整个过程是最耗时且最困难的过程，如下图所示。

![](/assets/ml-normal/1.png)

然后是机器学习模型应用于表格数据，表格数据是在机器学习或数据挖掘中表示数据的最常用方式。我们有一个数据表，包含不同数据样本（或X和标签y）的行。标签可以是单列或多列，具体取决于问题的类型。我们将用X表示数据，用y表示标签。

# 标签的类型

标签定义了问题，可以是不同的类型，如：

* 单列，二元值（分类问题，一个样本只属于一个类，只有两个类）
* 单列，实数值（回归问题，仅预测一个值）
* 多列，二元值（分类问题，一个样本属于一个类，但有两个以上的类）
* 多列，实数值（回归问题，多值预测）
* 多标签（分类问题，一个样本可以属于几个类）

# 评估指标

对于任何类型的机器学习问题，我们必须确定将如何评估我们的结果，要评估的指标或目标是什么？例如，如果存在倾斜的二分类问题，我们通常选择受试者工作特征曲线下的面积（ROCAUC或简称AUC）。在多标签或多分类问题的情况下，我们通常选择分类交叉熵或多分类对数损失，在回归问题的情况下选择均方误差。

我不会详细讨论不同的评估指标，我们可以有许多不同的类型，具体取决于问题本身。

# 机器学习库

要开始使用机器学习库，首先安装基本的和最重要的库，例如numpy和scipy。

查看和执行数据操作：pandas（[http://pandas.pydata.org/](http://pandas.pydata.org/)）
对于各种机器学习模型：scikit-learn（[http://scikit-learn.org/stable/](http://scikit-learn.org/stable/)）
最好的梯度提升库：xgboost（[https://github.com/dmlc/xgboost](https://github.com/dmlc/xgboost)）
对于神经网络：keras（[http://keras.io/](http://keras.io/)）
可视化数据：matplotlib（[http://matplotlib.org/](http://matplotlib.org/)）
监视进度：tqdm（[https://pypi.python.org/pypi/tqdm](https://pypi.python.org/pypi/tqdm)）

# 机器学习框架

2015年，我提出了一个自动机器学习框架，该框架目前仍在开发中，将于近期发布。在这篇文章中，相同的（基础）框架如下图所示：

![](/assets/ml-normal/2.png)

本图来自：A.Thakur和A.Krohn-Grimberghe，AutoCompete：机器学习竞赛框架，AutoMLWorkshop，2015年机器学习国际会议

在上面显示的框架中，粉红色的线代表最常用的路径。在我们提取并将数据缩减为表格格式后，我们可以继续构建机器学习模型。

第一步是确定问题，这可以通过查看标签来完成。必须知道问题是二分类、多分类、多标签分类还是回归问题。在我们确定问题后，我们将数据分成两个不同的部分，一个训练集和一个验证集，如下图所示。

![](/assets/ml-normal/3.png)

数据分解为训练集和验证集“必须”考虑标签类型。如果出现任何类型的分类问题，请使用分层分割。在python中，你可以很容易地使用scikit-learn。

![](/assets/ml-normal/4.png)

在回归任务的情况下，简单的K-Fold分割就足够了。然而，有些复杂的方法往往需要训练集和验证集保持相同的标签分布，这个问题将留给读者作为练习。

![](/assets/ml-normal/5.png)

上述示例中我已经选择全部数据的10％作为eval_size或验证集的大小，此值可以根据它们所具有的数据大小进行选择。

数据拆分完成后，请保留此数据不要动它。必须保存在训练集上应用的任何操作，然后将其应用于验证集。在任何情况下，验证集都不应该与培训集一起使用，这样做会产生非常好的评估分数并让用户感到满意，但是会构建一个严重过拟合的无用模型。

下一步是识别数据中的不同变量。我们处理的变量通常有三种类型，即数字变量、分类变量和包含文本的变量。让我们以流行的泰坦尼克号数据集（https://www.kaggle.com/c/titanic/data）为例。

![](/assets/ml-normal/6.png)

在这里，survival就是标签，我们已经将标签从前一步中的训练数据中分离出来了。变量pclass、sex、embarke具有不同的级别，因此它们是分类变量。像age、sibsp、parch等变量是数值变量。name是一个包含文本数据的变量，但我不认为它是预测生存的有用变量。

首先对于数值变量，这些变量不需要任何类型的处理，因此我们可以直接对这些变量应用规范化和机器学习模型。

有两种方法可以处理分类数据：

* 将分类数据转换为标签

![](/assets/ml-normal/7.png)

* 将标签转换为二元变量（one-hot编码）

![](/assets/ml-normal/8.png)

请记住先使用LabelEncoder将类别转换为数字，然后再应用OneHotEncoder。

因为泰坦尼克号的数据没有文本变量的好例子，所以让我们制定处理文本变量的一般规则。我们可以将所有文本变量合并为一个，然后使用一些对文本数据起作用的算法并将其转换为数字。

文本变量可以如下连接在一起：

![](/assets/ml-normal/9.png)

我们可以在变量上使用CountVectorizer或TfidfVectorizer：

![](/assets/ml-normal/10.png)

或

![](/assets/ml-normal/11.png)

TfidfVectorizer大多数时候比CountVectorizer性能都要好， TfidfVectorizer使用以下参数在很多情况下都有效。

![](/assets/ml-normal/12.png)

如果您只在训练集上应用这些vectorizer，请确保将其转储到磁盘中，以便稍后在验证集上使用它。

![](/assets/ml-normal/13.png)

接下来，我们来到stacker模块，stacker模块不是模型堆垛器，而是特征堆垛器。上述处理步骤之后的不同特征可以使用堆叠器模块进行组合。

![](/assets/ml-normal/14.png)

在通过使用numpy hstack或sparsehstack 进行进一步处理之前，您可以水平堆叠所有特征，具体取决于您是否拥有稠密或稀疏的特征。

![](/assets/ml-normal/15.png)

如果还有其他处理步骤如PCA或特征选择（我们将在本文的稍后部分讲解分解和特征选择），也可以通过FeatureUnion模块实现。

![](/assets/ml-normal/16.png)

一旦将这些特征堆叠在一起，我们可以开始应用机器学习模型。在这个阶段，你应该选择的模型应该集成基于树的模型。这些模型包括：

* RandomForestClassifier
* RandomForestRegressor
* ExtraTreesClassifier
* ExtraTreesRegressor
* XGBClassifier
* XGBRegressor

我们不能将线性模型应用于上述特征，因为它们没有归一化。要使用线性模型，可以使用scikit-learn中的Normalizer或StandardScaler。

这些归一化方法仅适用于稠密特征，如果应用于稀疏特征则不会给出非常好的结果。但可以在不使用均值的情况下在稀疏矩阵上应用StandardScaler（参数：with_mean= False）。

如果上述步骤给出了一个“好”的模型，我们可以去优化超参数，如果没有，我们需要继续下面的步骤并改进我们的模型。

![](/assets/ml-normal/17.png)

为了简单起见，我们将忽略LDA和QDA转换。对于高维数据，通常使用PCA来分解数据。对于图片以10-15个component开始，并且只要结果质量显着提高，就增加此数量。对于其他类型的数据，我们最初选择了50-60个component（只要我们能够处理得了数值数据，我们就倾向于避免PCA）。

![](/assets/ml-normal/18.png)

对于文本数据，在将文本转换为稀疏矩阵后，进行奇异值分解（SVD）。可以在scikit-learn中找到称为TruncatedSVD的SVD变体。

![](/assets/ml-normal/19.png)

通常用于TF-IDF或计数的SVD component的数量在120-200之间。以上任何数字都可能会提高性能，但不会实质性降低计算能力。

在进一步评估模型的性能之后，我们转向数据集的缩放，以便我们也可以评估线性模型。特征被归一化或缩放后可以被发送到机器学习模型或特征选择模块。

![](/assets/ml-normal/20.png)

有多种方法可以实现特征选择。最常见的方式之一是贪婪特征选择（向前或向后）。在贪婪特征选择中，我们选择一个特征，训练一个模型并根据固定评估指标评估模型的性能，我们一个接一个不断添加和删除特征，并在每一步记录模型的性能，最后我们选择评估得分最高的特征。以AUC作为评估指标的贪婪特征选择的一个实现可以在这里找到：https：//github.com/abhishekkrthakur/greedyFeatureSelection。必须指出的是，这种实现并不完美，必须根据要求进行修改。

其他更快的特征选择方法包括从模型中选择最佳特征。我们既可以查看logit模型的系数，也可以训练一个随机森林来选择最佳特征，然后在其他机器学习模型中使用这些特征。

![](/assets/ml-normal/21.png)

请记住，要保持较少的estimator和最少的超参数优化，以免过拟合。

使用梯度提升(GradientBoosting )也可以实现特征选择。我们在scikit-learn中推荐使用xgboost而不是GBM的实现，因为xgboost更快更灵活。

![](/assets/ml-normal/22.png)

我们也可以使用RandomForestClassifier / RandomForestRegressor和xgboost来进行稀疏数据集的特征选择。

从正稀疏数据集中选择特征的另一种流行方法是基于chi-2的特征选择，我们在scikit-learn中实现了这一功能。

![](/assets/ml-normal/23.png)

在这里，我们使用chi2和SelectKBest从数据中选择20个特征。这是我们想要优化以提高机器学习模型结果的超参数。

不要忘记导出在所有步骤中使用的任何种类的transformer，它们将被用于在验证集上的评估性能。

下一个（或中间）的主要步骤是模型选择+超参数优化。

![](/assets/ml-normal/24.png)

我们通常在选择机器学习模型的过程中使用以下算法：

* **分类**
	* Random Forest
	* GBM
	* Logistic Regression
	* Naive Bayes
	* Support Vector Machines
	* k-Nearest Neighbors

* **回归**
	* Random Forest
	* GBM
	* Linear Regression
	* Ridge
	* Lasso
	* SVR

我们应该优化哪些参数？如何选择趋近最佳的参数？这些是大多数人想到得到的几个问题。如果没有大量数据集上不同模型+参数的经验，就无法得到这些问题的答案，可能也有有经验的人不愿意分享他们的秘密。幸运的是，我也有相当多的经验，同时我愿意分享一些东西。

让我们分解超参数，使模型更智能：

![](/assets/ml-normal/25.png)

RS* =不能说完全适当的值，随机搜索这些超参数。

严格地说我的观点是，上述模型将超越其他模型，我们不需要评估任何其他模型。

再次记住保存transformer：

![](/assets/ml-normal/26.png)

并将它们分别应用于验证集：

![](/assets/ml-normal/27.png)

上述规则和框架在我处理的大多数数据集中都表现得非常好。当然，对于非常复杂的任务也可能是失败的。没有什么是完美的，我们继续改进我们学到的东西，就像机器学习一样。



