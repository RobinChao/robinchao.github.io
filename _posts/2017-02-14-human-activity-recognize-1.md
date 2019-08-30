---
layout: post
author: Robin
title: SVM学习 --- 基于智能手机传感器数据的人类行为识别（一）
tags: 机器学习 行为识别 SVM
categories:
  - 机器学习
  - SVM
--- 

> 本文已发布到infoQ，点击链接可查看。[infoQ](http://www.infoq.com/cn/articles/human-behavior-recognition-based-on-smart-phone-sensor-data)

人类行为识别的目的是通过一系列的观察，对人类的动作类型、行为模式进行分析和识别，并使用自然语言等方式对其进行描述的计算机技术。由于人类行为的复杂性和多样性，往往识别出的结果是多样性的，并且连带着行为类型的概率输出的。随着信息技术的发展，各种移动设备和可穿戴设备正在以加速度的方式增长，其性能和嵌入的传感器也变的多样化，例如：高清相机、光传感器、陀螺仪传感器、加速度传感器、GPS以及温度传感器等。各种各样的传感器都在时刻的记录着使用者的信息，这些记录信息不仅可以用于用户位置的预测，也可以进行用户行为的识别等。

本文使用了智能设备加速度传感器的数据，结合支持向量机的特性进行人类行为识别模型的设计和应用。 

<img src="/assets/har/sort.png"/> 

如上图所示，信号数据的采集来自于嵌入在智能手机中的加速度传感器，实验选用了人类日常行为中的六类常见行为，分别为：走路、慢跑、上楼梯、下楼梯、坐、站立，数据收集后，对数据进行特征抽取，抽取后的特征使用支持向量机的分类功能对特征进行分类，最后识别出人类的六类行为。

### 关于支持向量机（SVM）

**支持向量机**(Support Vector Machine)是Cortes和Vapnik于1995年首先提出的，它在解决小样本、非线性及高维模式识别中表现出许多特有的优势，并能够推广应用到函数拟合等其他机器学习问题中。SVM算法是基于间隔最大化的一种监督学习算法，包含线性和非线性两种模型，对于线性不可分问题，通常会加入核函数进行处理。

支持向量机本质上是一个二类分类方法，它的基本模型是定义在特征空间上的间隔最大化的线性分类器，间隔最大化使它有别于感知机。对于线性可分的训练集，感知机的分离超平面是不唯一的，会有无穷个，而支持向量机会对分离超平面增加约束条件，使得分类超平面唯一。

假设我们有一组分属于两类的二维点，分别用星和圆表示，这些点可以通过直线分割，我们需要找到一条最优的分割线:

* **找到正确的超平面（场景1）**：这里，我们有三个超平面(A、B、C)，我们需要找到正确的超平面来分割星和圆：

<img src="/assets/har/SVM_21.png"/>

我们的目的是**选择更好地分割两个类的超平面**，因此上图中可以看到超平面 **B** 已经能够完成分割的工作。

* **找到正确的超平面（场景2）**：同样有三个超平面(A、B、C)，我们需要找到正确的超平面来分割星和圆：

<img src="/assets/har/SVM_3.png"/> 

上图中，针对任意一个类，最大化最近的数据点和超平面之间的距离将有助于我们选择正确的超平面，这个距离称为**边距**，如下图：

<img src="/assets/har/SVM_4.png"/> 

可以看到，超平面**C**距离两个类别的边缘比A和B都要高，因此我们将超平面**C**定为最优的超平面。选择边距最高的超平面的另一个重要的原因是[鲁棒性](https://discuss.analyticsvidhya.com/t/what-does-robustness-means-in-svm-algorithm/5647)，假设我们选择最低边距的超平面，那么分类结果的错误率将会极大的升高。

* **找到正确的超平面（场景3）**：同样有三个超平面(A、B、C)，我们使用场景2的规则寻找正确的超平面来分割星和圆：

<img src="/assets/har/SVM_5.png"/>  

可能看到上图，第一印象最优的超平面是**B**，因为它比超平面A有更高的边距。但是，这里是一种意外情况，**支持向量机会选择在将边距最大化之前对类进行精确分类的超平面。**这里，超平面B具有分类误差，超平面A已经正确的分类，因此此情况下，最优超平面则是**A**。

* **能够分类两个类别（场景4）**：针对利群点情况，寻找最优超平面：

<img src="/assets/har/SVM_61.png"/>  

上图中，一个星出现在了圆所在的区域内，此星可称为利群点。但是支持向量机具有忽略异常点并找到具有最大边距的超平面的特征，因此，可以说，支持向量机是鲁棒性的。最终最优超平面如下图所示：

<img src="/assets/har/SVM_71.png"/> 

* **找到超平面并分类（场景5）**：以上的场景均是线性超平面。在下面的场景中，我们无法直接在两个类之间找到线性超平面，那么支持向量机如何分类这两个类呢？

<img src="/assets/har/SVM_8.png"/>  

支持向量机可以轻松的解决，它引入了一些附加的特性来解决此类问题。这里，我们添加一个新的特征**$z = x^2 + y^2$**。重新绘制坐标轴上的数据点如下：
 
<img src="/assets/har/SVM_9.png"/> 

在支持向量机中，已经很容易在这两个类直接找到线性超平面了，但是，出现的另一个重要的问题是，我们是否要手动处理这样的问题呢？当然不需要，在支持向量机中，有一个**[核函数](https://en.wikipedia.org/wiki/Kernel_method)**的技术，它会将低维空间的输入转换为高维空间，形成映射。由此会将某些不可分的问题转换为可分问题，主要用于一些非线性分类问题中。

当我们查看场景5中的超平面是，可能会如下图所示：

<img src="/assets/har/SVM_10.png"/>  


以上仅仅是关于支持向量机的一点介绍，支持向量机有这复杂的算法以及完备的证明，这里不再累述，可参考[Support_vector_machine](https://en.wikipedia.org/wiki/Support_vector_machine)查看学习。下面是一段支持向量机分类的3D演示视频，可以从视觉上体验一下支持向量机的分类特性：

<iframe src="/assets/har/svm_3d_play.mp4" frameborder="0" allowfullscreen></iframe>


对于支持向量机来说，比较有名的类库当属台湾大学林智仁(LinChih-Jen)教授所构建的[LIBSVM](http://www.csie.ntu.edu.tw/~cjlin/libsvm/)类库，由于LIBSVM程序小，运用灵活，输入参数少，并且是开源的，易于扩展，因此成为目前应用最多的支持向量机的库。 另外还提供了多种语言的接口，便于在不同的平台下使用，本文中使用的也是这个类库。 关于Mac下此类库的编译安装，请参考文档[Install libsvm on Mac OSX](http://macappstore.org/libsvm/)，本文会在Mac下进行训练数据预处理、模型训练、参数调优等，最终得到模型会使用在iOS项目中，当然该模型也可以使用在Android以及其他任何可以使用的地方。

针对支持向量机以及LIBSVM详细的介绍，可查看官方给出的文档：[PDF](http://www.csie.ntu.edu.tw/~cjlin/papers/guide/guide.pdf)  

### 传感器数据集

本文使用了 [WISDM (Wireless Sensor Data Mining)](http://www.cis.fordham.edu/wisdm/)  Lab 实验室公开的 Actitracker 的数据集。 WISDM 公开了两个数据集，一个是在实验室环境采集的；另一个是在真实使用场景中采集的，这里使用的是实验室环境采集的数据。

* 测试记录：1,098,207 条
* 测试人数：36 人
* 采样频率：20 Hz
* 行为类型：6 种
	* 走路
	* 慢跑
	* 上楼梯
	* 下楼梯
	* 坐
	* 站立
* 传感器类型：加速度
* 测试场景：手机放在衣兜里面

### 数据分析

从[实验室采集数据下载地址](http://www.cis.fordham.edu/wisdm/includes/datasets/latest/WISDM_ar_latest.tar.gz)下载数据集压缩包，解压后可以看到下面这些文件：

* readme.txt
* WISDM_ar_v1.1_raw_about.txt
* WISDM_ar_v1.1_trans_about.txt
* WISDM_ar_v1.1_raw.txt
* WISDM_ar_v1.1_transformed.arff

我们需要的是包含 RAW 数据的**WISDM_ar_v1.1_raw.txt** 文件，其他的是转换后的或者说明文件。先看看这些数据的分布情况：

``` python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from sklearn.metrics import precision_score, recall_score, f1_score, confusion_matrix, roc_curve, auc

if __name__ == "__main__":
    column_names = ['user-id', 'activity', 'timestamp', 'x-axis', 'y-axis', 'z-axis']
    df = pd.read_csv("WISDM_ar_v1.1_raw.txt", header=None, names=column_names)
    n = 10
    print df.head(n)
    subject = pd.DataFrame(df["user-id"].value_counts(), columns=["Count"])
    subject.index.names = ['Subject']
    print subject.head(n)
    activities = pd.DataFrame(df["activity"].value_counts(), columns=["Count"])
    activities.index.names = ['Activity']
    print activities.head(n)
    activity_of_subjects = pd.DataFrame(df.groupby("user-id")["activity"].value_counts())
    print activity_of_subjects.unstack().head(n)
    activity_of_subjects.unstack().plot(kind='bar', stacked=True, colormap='Blues', title="Distribution")
    plt.show()
```
**WISDM_ar_v1.1_raw.txt** 文件不是合法的 CSV 文件，每行后面有个 `;` 号，如果使用 `Pandas` 的 `read_csv `方法直接加载会出错，需要先将这些分号全部删除。
 
<img src="/assets/har/data_analytics.png"/> 

查看数据集各个行为的占比情况，绘制饼图如下：

<img src="/assets/har/data_pie.png" width="60%" height="60%"/>  

*可以看到此数据集是一个不平衡的数据集，但是这里暂时忽略其不平衡性。*

### 数据预处理

在LIBSVM的官方文档中可以看到，LIBSVM所使用的数据集是有严格的格式规范：

```
<label> <index1>:<value1> <index2>:<value2> ...
.
.
.
```

`<label>`：对于分类问题代表样本的类别，使用整数表示，支持多个类别；对于回归问题代表目标变量，可以是任意实数。

`<index1>:<value1>`：表示特征项。其中 `<index>` 代表特征项的编号，使用从 **1** 开始的整数表示，可以不连续；`<value>` 代表该特征项对应的特征值，使用实数表示。在实际的操作中，如果样本缺少某个特征项，可以直接省略，LIBSVM 会自动把该项的特征值赋为 **0**。

标签和每项特征之间使用`空格`分割，每行数据使用`\n`分割。

只有符合这样格式的数据，才能够被LIBSVM使用，否则会直接报错。这对准备好的数据，此类库还提供了一个**tools/checkdata.py**核查工具，以便核查数据集是否符合要求。针对特征的提取，为了简单，这里仅提取五类特征：

* 平均值
* 最大值
* 最小值
* 方差
* 组合三轴的加速度值 `math.sqrt(math.pow(acc_x, 2)+math.pow(acc_y, 2)+math.pow(acc_z, 2))`

了解了所需要的数据格式后，开始进行数据的预处理，并转换为所需要的格式文件。接下来分别将训练和测试数据集进行特征抽取并按照LIBSVM的数据格式重组，代码如下：

``` python
import ast
import math
import numpy as np 


FEATURE = ("mean", "max", "min", "std")
STATUS  = ("Sitting", "Walking", "Upstairs", "Downstairs", "Jogging", "Standing")

def preprocess(file_dir, Seg_granularity):
    gravity_data = []
    with open(file_dir) as f:
    	index = 0
        for line in f:
            clear_line = line.strip().lstrip().rstrip(';')
            raw_list = clear_line.split(',') 
            index = index + 1
            if len(raw_list) < 5:
            	continue
            status  = raw_list[1] 
            acc_x = float(raw_list[3])
            acc_y = float(raw_list[4])
            print index
            acc_z = float(raw_list[5])

            if acc_x == 0 or acc_y == 0 or acc_z == 0:
            	continue
            
            gravity = math.sqrt(math.pow(acc_x, 2)+math.pow(acc_y, 2)+math.pow(acc_z, 2))
            gravity_tuple = {"gravity": gravity, "status": status}
            gravity_data.append(gravity_tuple)

    # split data sample of gravity
    splited_data = []
    cur_cluster  = []
    counter      = 0
    last_status  = gravity_data[0]["status"]
    for gravity_tuple in gravity_data:
        if not (counter < Seg_granularity and gravity_tuple["status"] == last_status):
            seg_data = {"status": last_status, "values": cur_cluster}
            # print seg_data
            splited_data.append(seg_data)
            cur_cluster = []
            counter = 0
        cur_cluster.append(gravity_tuple["gravity"])
        last_status = gravity_tuple["status"]
        counter += 1
    # compute statistics of gravity data
    statistics_data = []
    for seg_data in splited_data:
        np_values = np.array(seg_data.pop("values"))
        seg_data["max"]  = np.amax(np_values)
        seg_data["min"]  = np.amin(np_values)
        seg_data["std"]  = np.std(np_values)
        seg_data["mean"] = np.mean(np_values)
        statistics_data.append(seg_data)
    # write statistics result into a file in format of LibSVM
    with open("WISDM_ar_v1.1_raw_svm.txt", "a") as the_file:
        for seg_data in statistics_data:
            row = str(STATUS.index(seg_data["status"])) + " " + \
                  str(FEATURE.index("mean")) + ":" + str(seg_data["mean"]) + " " + \
                  str(FEATURE.index("max")) + ":" + str(seg_data["max"]) + " " + \
                  str(FEATURE.index("min")) + ":" + str(seg_data["min"]) + " " + \
                  str(FEATURE.index("std")) + ":" + str(seg_data["std"]) + "\n"
            # print row
            the_file.write(row)        
    


if __name__ == "__main__":
    preprocess("WISDM_ar_v1.1_raw.txt", 100)
    pass  
```

成功转换后的数据格式形如：

```
.
.
.
5 0:9.73098373254 1:10.2899465499 2:9.30995703535 3:0.129482033438
5 0:9.74517171235 1:10.449291842 2:9.15706284788 3:0.161143714697
5 0:9.71565678822 1:10.4324206204 2:9.41070666847 3:0.136704694206
5 0:9.70622803003 1:9.7882020821 2:9.60614907234 3:0.0322246639852
5 0:9.74443440742 1:10.2915256401 2:9.28356073929 3:0.165543789197
0 0:9.28177794859 1:9.47500395778 2:8.92218583084 3:0.0700079500015
0 0:9.27218416165 1:9.40427562335 2:9.14709243421 3:0.0433805537826
0 0:9.27867211792 1:9.39755287296 2:9.1369415014 3:0.037533026091
0 0:9.27434585368 1:9.33462907672 2:9.21453200114 3:0.0263815511773
.
.
.

``` 

由于该数据集并未区分训练和测试数据集，因此为了最终的模型验证，首先需要分割该数据集为两份，分别进行训练和模型验证，分割方法就使用最简单的2\8原则，使用LIBSVM提供的工具`tools/subset.py`进行数据分割：

工具使用介绍：
```
Usage: subset.py [options] dataset subset_size [output1] [output2]

This script randomly selects a subset of the dataset.

options:
-s method : method of selection (default 0)
     0 -- stratified selection (classification only)
     1 -- random selection

output1 : the subset (optional)
output2 : rest of the data (optional)
If output1 is omitted, the subset will be printed on the screen.
```

使用工具进行数据分割：
``` shell
python subset.py -s 0 WISDM_ar_v1.1_raw_svm.txt 2190 raw_test.txt raw_train.txt
```
**
!! 注意 !! 
上面代码段中的**2190**就是subset.py工具子数据集的大小，该大小并不是文件的大小，而是根据原始文件中的行数进行2\8分后的行数。subset.py会随机抽取所设置行数的数据到指定的文件中。
**

完成后，我们就得到了训练数据集**raw_train.txt**和测试数据集**raw_test.txt**。


到此，所需要使用的数据集已经完全转换为LIBSVM所需要的格式，如果不放心数据格式，可以使用**tools/checkdata.py**工具进行检查。


### 模型创建与训练

在*关于支持向量机*部分，如果已经在Mac上安装好了libsvm，那么在你的命令行工具中输入**svm-train**，即可看到此命令的使用方式和参数说明，假设我们使用默认的参数进行模型训练：

``` shell
svm-train -b 1 raw_train.txt raw_trained.model
```
其中**-b** 的含义是probability_estimates，是否训练一个SVC或者SVR模型用于概率统计，设置为**1**，以便最终的模型评估使用。

训练过程的可能会消耗一点时间，主要在于所使用的训练数据集的大小，训练时的日志输出形如：

``` shell
.
.
.
optimization finished, #iter = 403
nu = 0.718897
obj = -478.778647, rho = -0.238736
nSV = 508, nBSV = 493
Total nSV = 508
*
optimization finished, #iter = 454
nu = 0.734417
obj = -491.057723, rho = -0.318206
nSV = 518, nBSV = 507
Total nSV = 518
*
optimization finished, #iter = 469
nu = 0.722888
obj = -604.608449, rho = -0.360926
nSV = 636, nBSV = 622
Total nSV = 4136
.
.
.
```
其中：`#iter` 是迭代次数，`nu` 是选择的核函数类型的参数，`obj` 为 SVM 文件转换为的二次规划求解得到的最小值，`rho` 为判决函数的偏置项 *b*，`nSV` 是标准支持向量个数（0 < a[i] < c），`nBSV` 是边界上的支持向量个数（a[i] = c），`Total nSV` 是支持向量总个数。

这样我们就得到了模型文件**raw_trained.model**，首先使用你所熟悉的文本编译工具打开此文件，让我们查看一下此文件中的内容：

``` shell
svm_type c_svc		//所选择的 svm 类型，默认为 c_svc
kernel_type rbf		//训练采用的核函数类型，此处为 RBF 核
gamma 0.333333		//RBF 核的 gamma 系数
nr_class 6		//类别数，此处为六元分类问题
total_sv 4136		//支持向量总个数
rho -0.369589 -0.28443 -0.352834 -0.852275 -0.831555 0.267266 0.158289 -0.777357 -0.725441 -0.271317 
-0.856933 -0.798849 -0.807448 -0.746674 -0.360926		//判决函数的偏置项 b
label 4 1 2 3 0 5		//类别标识
probA -3.11379 -3.0647 -3.2177 -5.78365 -5.55416 -2.30133 -2.26373 -6.05582 -5.99505 -1.07317 -4.50318 
-4.51436 -4.48257 -4.71033 -1.18804
probB 0.099704 -0.00543388 -0.240146 -0.43331 -1.01639 0.230949 0.342831 -0.249265 -0.817104 -0.0249471 
-0.209852 -0.691243 -0.0803133 -0.940074 0.272984
nr_sv 558 1224 880 825 325 324		//每个类的支持向量机的个数
SV
//以下为各个类的权系数及相应的支持向量
1 0 0 0 0 0:14.384883 1:24.418964 2:2.5636304 3:5.7143112 
1 1 1 0 0 0:11.867873 1:23.548919 2:4.5479318 3:4.5074937 
1 0 0 0 0 0:14.647238 1:24.192184 2:4.0759445 3:5.367968 
1 0 0 0 0 0:14.374831 1:24.286867 2:2.0045062 3:5.5710882 
1 0 0 0 0 0:14.099495 1:24.03442 2:2.42664 3:5.7580063 
1 0 0 0 0 0:14.313538 1:25.393975 2:1.9496137 3:5.6174387  
...
```
得到模型文件之后，首先要进行的就是模型的测试验证，还记得开始进行数据准备的时候，我们分割了训练和测试数据集吗？训练数据集进行了模型的训练，接下来就是测试数据集发挥作用的时候了。

验证模型，LIBSVM提供了另一个命令方法**svm-predict**，使用介绍如下：

``` shell
Usage: svm-predict [options] test_file model_file output_file
options:
-b probability_estimates: whether to predict probability estimates, 0 or 1 (default 0); 
   for one-class SVM only 0 is supported
-q : quiet mode (no outputs)
```

使用测试数据集进行模型验证：

```python
svm-predict -b 1 raw_test.txt raw_trained.model predict.out
```

执行此命令后，LIBSVM会进行识别预测，由于我们使用了**-b 1**参数，因此最终会输出各个类别的识别概率到*predict.out*文件中，并且会输出一个总体的正确率：

``` shell
Accuracy = 78.4932% (1719/2190) (classification)
```

可以看到此时我们训练的模型的识别正确率为*78.4932%*。

**predict.out** 文件内容形如：

``` shell
labels 4 1 2 3 0 5
4 0.996517 0.000246958 0.00128824 0.00123075 0.000414204 0.000303014
4 0.993033 0.000643327 0.00456298 0.00103339 0.000427387 0.000299934
1 0.0117052 0.773946 0.128394 0.0848292 0.00065714 0.0004682
1 0.0135437 0.484226 0.343907 0.156548 0.00105013 0.0007251
1 0.0117977 0.885448 0.0256842 0.0761578 0.000513167 0.000399136
3 0.00581106 0.380545 0.120613 0.490377 0.00179286 0.000861917
1 0.0117571 0.91544 0.0145561 0.0573158 0.000524352 0.000406782
1 0.0122297 0.811546 0.0824789 0.0924932 0.000704449 0.000547972
...
```

其中，第一行为表头，第一列是识别出的类别标签，后面依次跟着各个标签的识别概率。

那么问题来了，难道模型的识别正确率就只能到这个程度了吗？我们再次回顾**svm-train**命令，其中有很多的参数我们都使用了默认的设置，并没有进行特定的设置。通过查看LIBSVM官方的文档，发现竟然提供了**参数寻优**的工具**tools/grid.py**，通过此工具可以自动寻找训练数据集中的最优参数C系数和gamma系数，以在训练的时候使用。具体用法如下：

``` shell
Usage: grid.py [grid_options] [svm_options] dataset

grid_options :
-log2c {begin,end,step | "null"} : set the range of c (default -5,15,2)
    begin,end,step -- c_range = 2^{begin,...,begin+k*step,...,end}
    "null"         -- do not grid with c
-log2g {begin,end,step | "null"} : set the range of g (default 3,-15,-2)
    begin,end,step -- g_range = 2^{begin,...,begin+k*step,...,end}
    "null"         -- do not grid with g
-v n : n-fold cross validation (default 5)
-svmtrain pathname : set svm executable path and name
-gnuplot {pathname | "null"} :
    pathname -- set gnuplot executable path and name
    "null"   -- do not plot
-out {pathname | "null"} : (default dataset.out)
    pathname -- set output file path and name
    "null"   -- do not output file
-png pathname : set graphic output file path and name (default dataset.png)
-resume [pathname] : resume the grid task using an existing output file (default pathname is dataset.out)
    This is experimental. Try this option only if some parameters have been checked for the SAME data.

svm_options : additional options for svm-train
```

又是一堆的参数，但是不必担心，对于初学者来说，这里的大部分参数都可以不用设置，直接使用默认值即可，如果你需要查看参数寻优的过程，还需要安装[gnuplot](http://gnuplot.info/)并按照[官方说明](https://github.com/cjlin1/libsvm/tree/master/tools)配置。

``` python
python /tools/grid.py -b 1 raw_train.txt
```

执行此命令后，会不断的输出不同的C系数和gamma系数取值情况下的分类准确率，并在最后一行输出最优的参数选择：

``` shell
... 
[local] 13 -15 73.1217 (best c=8192.0, g=0.03125, rate=79.3446)
[local] 13 3 72.8477 (best c=8192.0, g=0.03125, rate=79.3446)
[local] 13 -9 77.8488 (best c=8192.0, g=0.03125, rate=79.3446)
[local] 13 -3 78.3741 (best c=8192.0, g=0.03125, rate=79.3446)
8192.0 0.03125 79.3446
```

并且会在当前目录下生成输出文件*raw_train.txt.out*和对应的图形文件*raw_train.txt.png*：
 
<img src="/assets/har/raw_train.txt.png"/> 


经过最优参数的寻找，最终给出了C系数为8192.0，gamma系数为0.03125的情况下，模型分类的准确率最高，为79.3446。

接下来我们再次使用**svm-train**方法，并设置当前最优C系数值和gamma系数值，重新训练我们的模型：

``` python
svm-train -b 1 -c  -g   raw_train.txt raw_bestP_trained.model
```

训练完成后，得到新的模型文件*raw_bestP_trained.model*，再次使用测试数据集进行验证：

``` python
svm-predict -b 1 raw_test.txt raw_bestP_trained.model bestP_predict.out
```

最终输出结果如下：

``` shell
Accuracy = 79.1324% (1733/2190) (classification)
```

可以看到模型的预测正确率明显提升了不少。上面的参数寻优仅仅是使用了默认的参数进行寻找，你也可以继续尝试设置各个参数进行参数寻优，以进一步提升模型识别正确率，这里不在进行进一步的参数寻优。

# 总结

可以看到SVM进行用户行为识别，可以得到较好的效果，本文中使用的数据是实验室数据，并且特征也仅仅提取了基本的几个，准确率即可达到79%以上，此方案可以继续进行优化，使用真实世界采集的数据，进行更加详细的特征准备，提高训练时的迭代次数等，进行模型重新训练优化，最终达到更好的分类效果。

在下一篇中，将在iOS平台下构建应用，并使用LIBSVM和本文中训练所得到的模型，进行准实时人类行为识别，敬请关注。
