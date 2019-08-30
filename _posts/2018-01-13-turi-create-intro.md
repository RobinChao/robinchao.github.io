---
layout: post
author: Robin
title: Apple开源机器学习框架 Turi Create 简介与实践
tags: 机器学习
categories:
  - 机器学习
cover: '/assets/turicreate/logo.png'
---

日前，苹果在 GitHub 平台上公布了 [Turi Create 框架](https://github.com/apple/turicreate)，苹果表示，这个框架旨在通过简化机器学习模型的开发，降低开发者构建模型的门槛。详细说明如下：

## Turi Create 概述

Turi Create简化了自定义机器学习模型的开发。你不需要成为机器学习的专家，即可为你的程序添加推荐，对象检测，图像分类，图像相似度识别或活动分类。

* **易于使用：**让你聚焦于任务而不是算法
* **可视化：**内置的流式可视化功能可以探索你的数据
* **灵活：**支持文本，图像，音频，视频和传感器数据
* **快速和可扩展性：**可在单台机器上处理大型数据集
* **易于准备配置：**模型导出到Core ML，即可用于iOS，macOS，watchOS和tvOS应用程序

使用Turi Create，你可以处理很多常见的场景：

* [推荐系统](https://github.com/apple/turicreate/blob/master/userguide/recommender/introduction.md)
* [图像分类](https://github.com/apple/turicreate/blob/master/userguide/image_classifier/introduction.md)
* [图像相似度检测](https://github.com/apple/turicreate/blob/master/userguide/image_similarity/introduction.md)
* [对象检测](https://github.com/apple/turicreate/blob/master/userguide/object_detection/introduction.md)
* [活动分类器](https://github.com/apple/turicreate/blob/master/userguide/activity_classifier/introduction.md)
* [文本分类器](https://github.com/apple/turicreate/blob/master/userguide/text_classifier/introduction.md)

你还可以使用基本的机器学习模型做成基于算法的工具包：

* [分类](https://github.com/apple/turicreate/blob/master/userguide/supervised-learning/classifier.md)
* [回归](https://github.com/apple/turicreate/blob/master/userguide/supervised-learning/regression.md)
* [图谱分析](https://github.com/apple/turicreate/blob/master/userguide/graph_analytics/intro.md)
* [聚类](https://github.com/apple/turicreate/blob/master/userguide/clustering/intro.md)
* [最近邻元素](https://github.com/apple/turicreate/blob/master/userguide/nearest_neighbors/nearest_neighbors.md)
* [主题模型](https://github.com/apple/turicreate/blob/master/userguide/text/intro.md)

### 支持的平台
Turi Create支持：

* macOS 10.12+
* Linux（依赖于glibc 2.12+）
* Windows 10（需要WSL）

### 系统要求
* Python 2.7（即将支持Python 3.5+）
* x86_64架构


### 安装
Linux不同变种的安装详细说明，参阅[LINUX_INSTALL.md](https://github.com/apple/turicreate/blob/master/LINUX_INSTALL.md)。常见的安装问题，参阅[INSTALL_ISSUES.md](https://github.com/apple/turicreate/blob/master/INSTALL_ISSUES.md)。

苹果官方推荐使用环境virtualenv，安装或建立Turi Create。请务必使用你的系统pip安装virtualenv。

``` shell
pip install virtualenv
```

安装Turi Create的方法参照[标准的python包安装步骤](https://packaging.python.org/installing/)。要创建一个名为venv的Python虚拟环境，请参照以下步骤:

```shell
# Create a Python virtual environment
cd ~
virtualenv venv
```

要激活新的虚拟环境并在此环境中安装Turi Create，请按照下列步骤操作：

```shell
# Active your virtual environment
source ~/venv/bin/activate

# Install Turi Create in the new virtual environment, pythonenv
(venv) pip install -U turicreate
```

另外，如果你使用的是[Anaconda](https://www.anaconda.com/what-is-anaconda/)，你可以使用它的虚拟环境：

```shell
conda create -n venv python=2.7 anaconda
source activate venv
```

在您的虚拟环境中安装Turi Create：

```shell
(venv) pip install -U turicreate
```

### GPU支持

Turi Create不一定需要GPU，但某些模型可以通过使用GPU加速。如果要在安装turicreate包后启用GPU支持，请执行以下步骤：

* 安装CUDA 8.0（[说明](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/)）
* 为CUDA 8.0安装cuDNN 5（[说明](https://developer.nvidia.com/cudnn)）

确保将CUDA库路径添加到**LD_LIBRARY_PATH**环境变量。通常情况下，这意味着将以下行添加到 **~/.bashrc**文件中：

```shell
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

如果你将cuDNN文件安装到单独的目录中，请确保单独添加它。下一步是卸载**mxnet**并安装启用CUDA的**mxnet-cu80**包：

```shell
(venv) pip uninstall -y mxnet
(venv) pip install mxnet-cu80==0.11.0
```

确保你安装的MXNet版本与turicreate依赖的版本相同（当前为0.11.0）。如果你在设置GPU时遇到困难，可参阅[MXNet安装说明](https://mxnet.incubator.apache.org/get_started/install.html)。

当然，除了上述的安装方式之外，你还可以从源码构建，如果想要从源码构建，可参阅[BUILD.md](https://github.com/apple/turicreate/blob/master/BUILD.md)中的详细说明。


## 猫狗大战

Turi Create可以训练定制的机器学习模型。这意味着可以开发可识别不同对象的模型，只要您使用大量图像训练模型即可。

这里我们以识别猫狗为例，建立一个自定义的模型，可以识别图像是猫还是狗。

#### 1. 下载猫和狗的图像

第一步是下载很多猫和狗的图像。这是必要的，因为我们将使用图像来训练我们的自定义模型。这里我使用的是[Kaggle Dogs vs. Cats](https://www.kaggle.com/c/dogs-vs-cats-redux-kernels-edition)的数据集。如果你觉得在这里单独下载麻烦，可直接在[Kaggle Cats and Dogs Dataset](https://download.microsoft.com/download/3/E/1/3E1C3F21-ECDB-4869-8368-6DEBA77B919F/kagglecatsanddogs_3367a.zip)中下载全量数据集。

![](/assets/turicreate/get_data.png)
 
下载好图像数据集之后，我们解压到对应的文件夹内备用。

#### 2. 标记数据、训练模型

在开始训练我们的模型前，首先需要对每一张图像标记其为‘cat’还是‘dog’。幸运的是Turi Create提供了标记基于不同文件夹的图像的功能，具体代码如下：

```python
import turicreate as tc

# load the images
data = tc.image_analysis.load_images('PetImages', with_path = True)
data['label'] = data['path'].apply(lambda path:'Dog' if 'Dog' in path else 'Cat')
print(data.groupby('label',[tc.aggregate.COUNT]))
# save the data
data.save('cats-dogs.sframe')

data.explore()
```

标记完成后，可以看到数据集的全局情况：

![](/assets/turicreate/data_taging.png)

上述代码的最后一行 *data.explore()*，会自动打开Turi Create的图像可视化查看工具，在这里你可以看到每张图像以及相应的标记，也是一种核查标记是否正确的方式。

![](/assets/turicreate/image_visualizer.png)


数据集准备完成后，就可以进行模型的训练了。在训练的时候，会将数据集按照‘二八原则’进行训练集和测试集划分，然后进行模型训练：

```python
import turicreate as tc

# load the data
data = tc.SFrame('cats-dogs.sframe')
# random split data 
train_data, test_data = data.random_split(0.8)
# train model
model = tc.image_classifier.create(train_data, target='label')
# test model
predictions = model.predict(test_data)
# get model metrics
metrics = model.evaluate(test_data)
# print test accuracy
print(metrics['accuracy'])
# save .model
model.save('mymodel.model')
# export CoreML model
model.export_coreml('CatsAndDogs.mlmodel')
```

训练的过程可能会花一点时间，长短取决对机器的配置。在训练的过程中，Turi Create会打印出每一步执行的动作，如下：

![](/assets/turicreate/training1.png)
……
![](/assets/turicreate/training2.png)

最终我们得到的正确率为：**0.986954749287**，但看这个结果还不错。并且已经导出了**mymodel.model**和支持Core ML 的 **CatsAndDogs.mlmodel**。

#### 3. 移植模型到iOS应用程序

有了**CatsAndDogs.mlmodel**模型文件后，我们就可以将其移植到iOS应用程序中了。关于如何集成，可参考Apple官方的图像识别例子，这里不再陈述：

* [Integrating a Core ML Model into Your App](https://developer.apple.com/documentation/coreml/integrating_a_core_ml_model_into_your_app)
* [Classifying Images with Vision and Core ML](https://developer.apple.com/documentation/vision/classifying_images_with_vision_and_core_ml)


## 总结

本篇内容对Apple公开的Turi Create机器学习框架进行了简单的介绍，并实践了Turi Create在图像识别方面的一些基本用法。Turi Create不仅仅能用于图像识别，在其他方面能有很好的表现，目前Turi Create还刚开源不久，相信在社区的力量下，会带来不同的功能和体验，拭目以待。

