---
layout: post
author: Robin
title: 如何使用Turi Create进行人类行为识别
tags: 机器学习
categories:
  - 机器学习
---

**行为识别**也可称为活动分类器，是模式识别技术的一种，是特征提取和行为分类的结合。

它通过运动传感器收集序列性数据，进行特征提取并输入到预定义的预测模型中，识别出其所对应的动作的任务。此类传感器有很多，例如加速度计、陀螺仪等等。而对应的应用也很多，例如使用集成在手表中的加速度计数据来计算游泳的圈数，使用手机中的陀螺仪数据识别手势，并在特定手势时打开蓝牙控制的灯光，或者使用自定义的手势来为创建一个快捷方式等等。Turi Create 中的活动分类器创建了一个深度学习模型，能够检测传感器数据中的时间特征，能够很好地适应活动分类的任务。在我们深入模型架构之前，让我们看一个可行的例子。

## 示例简介

在这个例子中，我们将使用手持设备的加速度计和陀螺仪数据创建一个活动分类模型，以识别用户的物理活动行为。这里我们将使用公开的数据集 [HAPT 实验](http://archive.ics.uci.edu/ml/datasets/Smartphone-Based+Recognition+of+Human+Activities+and+Postural+Transitions)的相关数据，这些数据中包含多个用户的会话记录，每个用户执行特定的身体活动。执行的身体活动包括：步行、上楼梯、下楼梯、坐、站和躺。

传感器的数据可以以不同的频率进行收集。在HAPT数据集中，传感器以 50Hz 的采样频率进行的采集，也就是说每秒钟会采集50个点。但是在大部分的应用程序中，都会以更长的时间间隔来展示预测输出，因此我们会通过参数 ***prediction_window*** 来控制预测率的输出。例如，如果我们希望每5秒钟产生一次预测，并且传感器以50Hz进行采样，则我们将预测窗口设置为250（5秒 * 每秒50个采样）。

以下是HAPT数据集​​中单个会话3秒的 “步行” 数据的示例：

![](/assets/turicreate/walking.png)

以下是HAPT数据集​​中单个会话3秒的 “站立” 数据的示例：

![](/assets/turicreate/sitting.png)


活动分类器的初级目标是区分这些数据样本，但是在开始之前，我们需要对这些数据进行预处理，以获取数据的SFrame结构体，作为Turi Create活动分类器的输入。



## 数据预处理

在这部分，我们将会对 [HAPT 实验](http://archive.ics.uci.edu/ml/datasets/Smartphone-Based+Recognition+of+Human+Activities+and+Postural+Transitions) 的数据转换为Turi Create活动分类器所期望的SFrame格式。

首先，需要下载数据集的zip格式数据文件，你可以点击 [这里](http://archive.ics.uci.edu/ml/machine-learning-databases/00341/HAPT%20Data%20Set.zip) 直接下载。在下面的代码中，我们假定zip格式的数据被解压缩到了 **HAPT Data Set** 的文件夹中。文件夹中包含三种类型的文件 --- 包含每个实验所执行的活动的文件、包含收集的加速度计样本的文件和收集陀螺仪数据的样本文件。


其中，文件 `labels.txt` 包含为每个实验所执行的活动。每个活动标签是通过样本的索引指定的。例如，在实验1中，受试者在第250次收集的样品和第1232次收集的样品之间进行了第5次活动。活动标签被编码为 1 到 6 的数字。我们将在本节最后转换这些数字为字符串。首先，我们加载 `labels.txt` 内容，转换到SFrame中，并定义一个函数来查找给定样本索引所对应的标签。

```python
# import Turi Create
import turicreate as tc

# define data directory (you need use yours directory path)
data_dir = '../HAPT Data Set/RawData/'

# define find label for containing interval
def find_label_for_containing_interval(intervals, index):
    containing_interval = intervals[:, 0][(intervals[:, 1] <= index) & (index <= intervals[:, 2])]
    if len(containing_interval) == 1:
        return containing_interval[0]

# load labels
labels = tc.SFrame.read_csv(data_dir + 'labels.txt', delimiter=' ', header=False, verbose=False)
# rename CSV header
labels = labels.rename({'X1': 'exp_id', 'X2': 'user_id', 'X3': 'activity_id', 'X4': 'start', 'X5': 'end'})
print labels
```

如果运行正常，则输出如下：

```
+--------+---------+-------------+-------+------+
| exp_id | user_id | activity_id | start | end  |
+--------+---------+-------------+-------+------+
|   1    |    1    |      5      |  250  | 1232 |
|   1    |    1    |      7      |  1233 | 1392 |
|   1    |    1    |      4      |  1393 | 2194 |
|   1    |    1    |      8      |  2195 | 2359 |
|   1    |    1    |      5      |  2360 | 3374 |
|   1    |    1    |      11     |  3375 | 3662 |
|   1    |    1    |      6      |  3663 | 4538 |
|   1    |    1    |      10     |  4539 | 4735 |
|   1    |    1    |      4      |  4736 | 5667 |
|   1    |    1    |      9      |  5668 | 5859 |
+--------+---------+-------------+-------+------+
[1214 rows x 5 columns]
Note: Only the head of the SFrame is printed.
You can use print_rows(num_rows=m, num_columns=n) to print more rows and columns.
```

接下来，我们需要从实验数据中获取加速度计和陀螺仪数据。对于每一次实验，每种传感器数据都存储在分开的文件中。接下来我们会将加载所有实验中的加速度计和陀螺仪数据到单独的一个SFrame中。在加载收集的样本时，我们还使用之前定义的 `find_label_for_containing_interval` 函数计算每个样本的标签。最终的SFrame包含一个名为exp_id的列来标识每个唯一的会话。

```python
from glob import  glob

acc_files = glob(data_dir + 'acc_*.txt')
gyro_files = glob(data_dir + 'gyro_*.txt')

# load datas
data = tc.SFrame()
files = zip(sorted(acc_files), sorted(gyro_files))
for acc_file, gyro_file in files:
    exp_id = int(acc_file.split('_')[1][-2:]) 

    # load accel data
    sf = tc.SFrame.read_csv(acc_file, delimiter=' ', header=False, verbose=False)
    sf = sf.rename({'X1': 'acc_x', 'X2': 'acc_y', 'X3': 'acc_z'})
    sf['exp_id'] = exp_id 

    # load gyro data
    gyro_sf = tc.SFrame.read_csv(gyro_file, delimiter=' ', header=False, verbose=False)
    gyro_sf = gyro_sf.rename({'X1': 'gyro_x', 'X2': 'gyro_y', 'X3': 'gyro_z'})
    sf = sf.add_columns(gyro_sf)

    # calc labels
    exp_labels = labels[labels['exp_id'] == exp_id][['activity_id', 'start', 'end']].to_numpy()
    sf = sf.add_row_number()
    sf['activity_id'] = sf['id'].apply(lambda x: find_label_for_containing_interval(exp_labels, x))
    sf = sf.remove_columns(['id'])

    data = data.append(sf)
```

```
+----------------+------------------+----------------+--------+---------+
|     acc_x      |      acc_y       |     acc_z      | exp_id | user_id |
+----------------+------------------+----------------+--------+---------+
| 0.918055589877 | -0.112499999424  | 0.509722251429 |   1    |    1    |
| 0.91111113046  | -0.0930555616826 | 0.537500040471 |   1    |    1    |
| 0.88194449816  | -0.0861111144223 | 0.513888927079 |   1    |    1    |
| 0.88194449816  | -0.0861111144223 | 0.513888927079 |   1    |    1    |
| 0.879166714393 | -0.100000002865  | 0.50555557578  |   1    |    1    |
| 0.888888957576 |  -0.10555556432  | 0.512500035196 |   1    |    1    |
| 0.862500011794 | -0.101388894748  | 0.509722251429 |   1    |    1    |
| 0.861111119911 | -0.104166672437  | 0.50138890013  |   1    |    1    |
| 0.854166660495 |  -0.10833333593  | 0.527777797288 |   1    |    1    |
| 0.851388876728 | -0.101388894748  | 0.552777802563 |   1    |    1    |
+----------------+------------------+----------------+--------+---------+
+------------------+------------------+------------------+-------------+
|      gyro_x      |      gyro_y      |      gyro_z      | activity_id |
+------------------+------------------+------------------+-------------+
| -0.0549778714776 | -0.0696386396885 | -0.0308486949652 |     None    |
| -0.0125227374956 | 0.0192422550172  | -0.0384845100343 |     None    |
| -0.0235183127224 |  0.276416510344  | 0.00641408516094 |     None    |
| -0.0934623852372 |  0.367740869522  | 0.00122173049022 |     None    |
| -0.124311074615  |  0.476780325174  | -0.0229074470699 |     None    |
| -0.100487336516  |  0.519846320152  | -0.0675006061792 |     None    |
| -0.149356558919  |  0.481056392193  | -0.0925460830331 |     None    |
| -0.211053937674  |  0.389121174812  |  -0.07483099401  |     None    |
| -0.222354948521  |  0.267864406109  | -0.0519235469401 |     None    |
| -0.173791155219  |  0.207083314657  | -0.0320704244077 |     None    |
+------------------+------------------+------------------+-------------+
[1122772 rows x 9 columns]
```

最后，我们将标签数字格式化为更加直观的字符串形式，并保存处理后的数据到SFrame，如下：

```python

target_map = {
    1.: 'walking',
    2.: 'climbing_upstairs',
    3.: 'climbing_downstairs',
    4.: 'sitting',
    5.: 'standing',
    6.: 'laying'
}

# Use the same labels used in the experiment
data = data.filter_by(target_map.keys(), 'activity_id')
data['activity'] = data['activity_id'].apply(lambda x: target_map[x])
data  = data.remove_column('activity_id')

data.save('hapt_data.sframe')
```

```
+---------------+-----------------+-----------------+--------+---------+
|     acc_x     |      acc_y      |      acc_z      | exp_id | user_id |
+---------------+-----------------+-----------------+--------+---------+
| 1.02083339474 | -0.125000002062 |  0.10555556432  |   1    |    1    |
| 1.02500007039 | -0.125000002062 |  0.101388894748 |   1    |    1    |
| 1.02083339474 | -0.125000002062 |  0.104166672437 |   1    |    1    |
| 1.01666671909 | -0.125000002062 |  0.10833333593  |   1    |    1    |
| 1.01805561098 | -0.127777785828 |  0.10833333593  |   1    |    1    |
| 1.01805561098 | -0.129166665555 |  0.104166672437 |   1    |    1    |
| 1.01944450286 | -0.125000002062 |  0.101388894748 |   1    |    1    |
| 1.01666671909 | -0.123611110178 | 0.0972222251764 |   1    |    1    |
| 1.02083339474 | -0.127777785828 | 0.0986111170596 |   1    |    1    |
| 1.01944450286 | -0.115277783191 | 0.0944444474879 |   1    |    1    |
+---------------+-----------------+-----------------+--------+---------+
+--------------------+-------------------+-------------------+----------+
|       gyro_x       |       gyro_y      |       gyro_z      | activity |
+--------------------+-------------------+-------------------+----------+
| -0.00274889357388  | -0.00427605677396 |  0.00274889357388 | standing |
| -0.000305432622554 | -0.00213802838698 |  0.00610865233466 | standing |
|  0.0122173046693   | 0.000916297896765 | -0.00733038317412 | standing |
|  0.0113010071218   | -0.00183259579353 | -0.00641408516094 | standing |
|  0.0109955742955   | -0.00152716308367 | -0.00488692196086 | standing |
|  0.00916297826916  | -0.00305432616733 |   0.010079276748  | standing |
|   0.010079276748   | -0.00366519158706 | 0.000305432622554 | standing |
|  0.0137444678694   |  -0.0149661982432 |  0.00427605677396 | standing |
|  0.00977384392172  | -0.00641408516094 | 0.000305432622554 | standing |
|  0.0164933614433   |  0.00366519158706 |  0.00335975876078 | standing |
+--------------------+-------------------+-------------------+----------+
[748406 rows x 9 columns]
```

这样数据的预处理就结束了，但是有一个问题，为什么要这样来处理数据呢？接下来我们详细来看看。

## 数据预处理理论介绍

在本节中，我们将介绍活动分类器的输入数据格式以及可用的不同输出格式。

#### 输入数据格式

活动分类器是根据一段特定时间内以特定的频率收集的，来自不同传感器的的数据创建的。**Turi Create的活动分类器中，所有传感器都以相同的频率进行采样。**例如，在HAPT实验中，数据包含三轴加速度和三轴陀螺仪，在每个时间点，会产生6个值（特征）。每个传感器的样本收集频率为50Hz，也就是每秒钟收集50个样本点。下图显示了HAPT实验中从单个受试者收集的3秒步行数据：

![](/assets/turicreate/walking.png)

而传感器的采样频率取决于被分类的活动和各种实际情况的限制。例如，尝试检测极小的活动（比如手指抖动），则可能需要较高的采样频率，而较低的频率则可能需要检测那些比较粗糙的活动（比如游泳），更进一步来说，还要考虑设备的电量问题和模型的构建时长问题等。高频率的采样行为，则需要更多传感器和其数据捕获，这会导致更高的电量消耗和更大的数据量，增加了模型的复杂性和创建时长等。

一般情况下，使用活动分类器的应用程序都会根据不同的活动来为用户提供比传感器采样率更慢的预测。例如，计步器可能需要每秒钟进行一次预测，而为了检测睡眠，可能每分钟才进行一次预测。在构建模型的时候，重要的是要提供和期望的预测速率相同的标签，与单个标签关联的传感器样本的数量被称之为**预测窗口**。活动分类器就是使用预测窗口来确定预测速率，即在每个预测窗口样本之后进行预测。对于HAPT数据集来说，我们使用的**prediction_window**是50，当传感器以50Hz的频率采样时，每秒产生一个预测。

从对象的单个记录产生的每组连续的样本称为**会话**。一个会话可以包含多个活动的示例，会话并不需要包含所有活动或者具有相同的长度。活动分类器的输入数据必须包含一个列向数据，以便将每个样本唯一地分配给一个会话。Turi Create中的活动分类器期望与每个会话id的数据样本关联并按照时间升序排序。

一下是HAPT数据集经过预处理后，得到的活动分类器所期望的输入SFrame格式示例。该示例包含2个会话，有*exp_id*区分，在这个例子中，第一次会话仅仅包含步行样本，而第二个会话则包含站立和坐着的样本。


```
+--------+----------+----------+-----------+----------+-----------+-----------+-----------+
| exp_id | activity |  acc_x   |   acc_y   |  acc_z   |   gyro_x  |   gyro_y  |   gyro_z  |
+--------+----------+----------+-----------+----------+-----------+-----------+-----------+
|   1    | walking  | 0.708333 | -0.197222 | 0.095833 | -0.751059 |  0.345444 |  0.038179 |
|   1    | walking  | 0.756944 | -0.173611 | 0.169444 | -0.545503 |  0.218995 |  0.046426 |
|   1    | walking  | 0.902778 | -0.169444 | 0.147222 | -0.465785 |  0.440128 | -0.045815 |
|   1    | walking  | 0.970833 | -0.183333 | 0.118056 | -0.357662 |  0.503964 | -0.206472 |
|   1    | walking  | 0.972222 | -0.176389 | 0.166667 | -0.312763 |  0.64263  | -0.309709 |
|   2    | standing | 1.036111 | -0.290278 | 0.130556 |  0.039095 | -0.021075 |  0.034208 |
|   2    | standing | 1.047222 | -0.252778 |   0.15   |  0.135612 |  0.015272 | -0.045815 |
|   2    | standing |  1.0375  | -0.209722 | 0.152778 |  0.171042 |  0.009468 | -0.094073 |
|   2    | standing | 1.026389 |  -0.1875  | 0.148611 |  0.210138 | -0.039706 | -0.094073 |
|   2    | sitting  | 1.013889 | -0.065278 | 0.127778 | -0.020464 | -0.142332 |  0.091324 |
|   2    | sitting  | 1.005556 | -0.058333 | 0.127778 | -0.059254 | -0.138972 |  0.055589 |
|   2    | sitting  |   1.0    | -0.070833 | 0.147222 | -0.058948 | -0.124922 |  0.026878 |
+--------+----------+----------+-----------+----------+-----------+-----------+-----------+
[12 rows x 8 columns]
```

在这个例子中，如果*prediction_window*设置为2，那么会话中的没两行数据将被作为预测输入，会话结束的时候，预测窗口中数据行数小于预测窗口行数也会产生预测。预测窗口 2 将产生针对*exp_id 1* 的 3 个预测和针对 *exp_id 2* 的 4 个预测，而预测窗口 5 将针对 *exp_id 1*产生单个预测，并针对 *exp_id 2* 产生 2 个预测。

#### 预测频率

之前有提到过，活动分类器的预测频率是有预测窗口参数**prediction_window**确定的。因此，会话中的每个预测窗口行都会产生一个预测。对于上述HAPT数据集来说，将预测串钩设置为50的含义为，没50个样本产生一个预测。

```python
model.predict(walking_3_sec, output_frequency='per_window')
```

```
+---------------+--------+---------+
| prediction_id | exp_id |  class  |
+---------------+--------+---------+
|       0       |   1    | walking |
|       1       |   1    | walking |
|       2       |   1    | walking |
+---------------+--------+---------+
[3 rows x 3 columns]
```

然而，在许多机器学习的工作流程中，通常使用来自一个模型的预测作为进一步分析和建模的输入。在这种情况下，返回每行输入数据的预测可能会更有益。我们可以通过将output_frequency参数设置为**per_row**来要求模型执行此操作。

```python
model.predict(walking_3_sec, output_frequency='per_row')
```

```
dtype: str
Rows: 150
['walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', 'walking', ... ]
```

这些预测是通过在与所述窗口相关联的所有样本上复制每个预测窗口的每个预测而产生的。

## 模型训练

上面讲述了大量的数据预处理知识，也已经将数据处理为活动分类器所期望的格式和结构，下面我们来使用数据进行模型的训练。

```python
import  turicreate as tc

# load sessions from preprocessed data
data = tc.SFrame('hapt_data.sframe')

# train/test split by recording sessions
train, test = tc.activity_classifier.util.random_split_by_session(data, session_id='exp_id', fraction=0.8)

# create an activity classifier
model = tc.activity_classifier.create(train, 
	session_id='exp_id', 
	target='activity', 
	prediction_window=50)

# evaluate the model and save result into dictionary
metrics = model.evaluate(test)
print (metrics['accuracy'])
```

训练的执行过程，会有如下的日志输出：

```
Pre-processing 575999 samples...
Using sequences of size 1000 for model creation.
Processed a total of 47 sessions.
Iteration: 0001
	Train loss    : 1.384639084 	Train accuracy: 0.423688752
Iteration: 0002
	Train loss    : 0.975227836 	Train accuracy: 0.604033018
Iteration: 0003
	Train loss    : 0.858876649 	Train accuracy: 0.658348667
Iteration: 0004
	Train loss    : 0.747760415 	Train accuracy: 0.696624932
Iteration: 0005
	Train loss    : 0.717178401 	Train accuracy: 0.710401664
Iteration: 0006
	Train loss    : 0.708376906 	Train accuracy: 0.720765597
Iteration: 0007
	Train loss    : 0.727093298 	Train accuracy: 0.712437319
Iteration: 0008
	Train loss    : 0.701619904 	Train accuracy: 0.730136608
Iteration: 0009
	Train loss    : 0.719597752 	Train accuracy: 0.713592718
Iteration: 0010
	Train loss    : 0.618533716 	Train accuracy: 0.766228394
Training complete
Total Time Spent: 12.3062s
0.804323490346
```

可以看到，默认情况下，迭代仅仅进行了 10 次，我们可以通过参数**max_iterations**来设置迭代次数，例如：

```python
model = tc.activity_classifier.create(train, 
	session_id='exp_id', 
	target='activity', 
	prediction_window=50, 
	max_iterations=20)
```

此时得到的准确率会提升到：0.835319045889。因此一个合适的迭代次数设置也是必须的。

训练完成后，我们可以将模型数据保存下来，以待下次使用。如：

```python
# save the model for later use in Turi Create
model.save('mymodel.model')
```

另外，也可以直接导出到Core ML所支持的模型文件格式，如下：

```python
# export for use in Core ML
model.export_coreml('MyActivityClassifier.mlmodel')
```

由于我们已经创建了采样频率为50Hz的模型，并将prediction_window设置为50，我们将得到每秒一个预测。接下来，我们使用文章开头给出的3秒的步行数据来测试一下：

```python
# load saved model
activityClassifier = tc.load_model('mymodel.model')

# load sessions from preprocessed data
data = tc.SFrame('hapt_data.sframe')

# filter the walking data in 3 sec
walking_3_sec = data[(data['activity'] == 'walking') & (data['exp_id'] == 1)][1000:1150]

# do predict
predicts = activityClassifier.predict(walking_3_sec, output_frequency='per_window')
print predicts
```

```
+---------------+--------+---------+
| prediction_id | exp_id |  class  |
+---------------+--------+---------+
|       0       |   1    | walking |
|       1       |   1    | walking |
|       2       |   1    | walking |
+---------------+--------+---------+
[3 rows x 3 columns]
```

至此，我们已经看到了如何使用传感器数据快速构建一个活动分类器了，接下来我们来看看如何在iOS中使用Core ML来使用此活动分类器。


## 部署到Core ML

在上一节中，我们已经将训练的模型导出了Core ML所支持的文件格式*mlmodel*格式了。而Core ML是iOS平台上进行快速机器学习模型集成和使用的框架，使用简单而且快速，我们将使用Swift语言编写整个集成部署代码。

首先，创建一个空的工程项目，并制定语言使用Swift。导入上一步的**MyActivityClassifier.mlmodel**文件到Xcode项目，Xcode会自动生成一些相关的API代码和参数说明：

![](/assets/turicreate/model_in_xcode.png)

更多此方面的信息，可参考[Core ML 官方文档](https://developer.apple.com/documentation/coreml/integrating_a_core_ml_model_into_your_app)。

从上图中可以看到，整个模型的数据交互分为两大部分，一部分为输入（inputs），另一部分为输出（outputs）：

#### 模型输入

* **features：**特征数组，其长度为*prediction_window*，宽度为特征的数量。其中包含传感器的读数，这些读数已经进行了汇总。
* **hiddenIn：**模型中LSTM recurrent层的输入状态。当开始一个新的会话是初始化为0，否则应该用前一个预测的*hiddenOut*进行输入。
* **cellIn：**模型中LSTM recurrent层的神经元输入状态。当开始一个新的会话是初始化为0，否则应该用前一个预测的*cellOut*进行输入。

#### 模型输出

* **activityProbability：**概率字典。其中key为每种标签，也就是活动类型，value为属于该类别的概率，其值范围是[0.0,1.0]。
* **activity：**代表预测结果的字符串。该值和*activityProbability：*中概率最高的那种活动类别相对应。
* **hiddenOut：**模型中LSTM recurrent层的输出状态。这个输出应该保存下来，并在下次预测调用时输入到模型的*hiddenIn*中。
* **cellOut：**模型中LSTM recurrent层的神经元输出状态。这个输出应该保存下来，并在下次预测调用时输入到模型的*cellIn*中。

关于模型详细的结构信息以及是如何工作的，可以参考[如果工作的？](https://github.com/apple/turicreate/blob/master/userguide/activity_classifier/how-it-works.md)。

### 在应用程序中应用Core ML模型

在iOS/watchOS应用中部署活动分类模型涉及3个基本步骤：

1. 启用相关传感器，并将其设置为所需的频率。
2. 将来自传感器的读数汇总到一个*prediction_window*长阵列中。
3. 当数组阵列变满时，调用模型的*prediction()*方法来获得预测的活动。

#### 创建用于汇总输入的数组

活动分类器模型期望接收的数据是包含传感器数据并符合*prediction_window*读数的数组。

应用程序需要将传感器的读数汇合成一个尺寸为 **1 x prediction_window x number_of_features**的 *MLMultiArray*。

另外，应用程序还需要保存每层最后的*hiddenOut*和*cellOut*输出，以便在下一次预测中输入到模型中。

首先我们定义个结构体，用来设定相关的数值类型参数：

```swift
struct ModelConstants {
    static let numOfFeatures = 6
    static let predictionWindowSize = 50
    static let sensorsUpdateInterval = 1.0 / 50.0
    static let hiddenInLength = 200
    static let hiddenCellInLength = 200
}
```

之后，初始化模型对象：

```swift
let activityClassificationModel = MyActivityClassifier()
```

我们还需要初始化一些变量，包括数据数组、当前窗口大小、最后*hiddenOut*和*cellOut*输出变量：

```swift
var currentIndexInPredictionWindow = 0
let predictionWindowDataArray = try? MLMultiArray(
       shape: [1, ModelConstants.predictionWindowSize, ModelConstants.numOfFeatures] as [NSNumber],
       dataType: MLMultiArrayDataType.double)
var lastHiddenOutput = try? MLMultiArray(
       shape: [ModelConstants.hiddenInLength as NSNumber],
       dataType: MLMultiArrayDataType.double)
var lastHiddenCellOutput = try? MLMultiArray(
       shape: [ModelConstants.hiddenCellInLength as NSNumber],
       dataType: MLMultiArrayDataType.double)
```

#### 启用CoreMotion传感器

我们需要启用加速计和陀螺仪传感器，将它们设置为所需的更新间隔并设置我们的处理程序块：

更多关于CoreMotion传感器的内容，可参考[CoreMotion文档](https://developer.apple.com/documentation/coremotion)。

```swift
let motionManager: CMMotionManager? = CMMotionManager()

func startMotionSensor() {
    guard let motionManager = motionManager, motionManager.isAccelerometerAvailable && motionManager.isGyroAvailable else { return }
    motionManager.accelerometerUpdateInterval = TimeInterval(ModelConstants.sensorsUpdateInterval)
    motionManager.gyroUpdateInterval = TimeInterval(ModelConstants.sensorsUpdateInterval)
        
    // Accelerometer sensor
    motionManager.startAccelerometerUpdates(to: .main) { (accelerometerData, error) in
        guard let accelerometerData = accelerometerData else {return}
            
        // add the current acc data sample to the data array
        
    }
    // Gyro sensor
    motionManager.startGyroUpdates(to: .main) { (gyroData, error) in
        guard let gyroData = gyroData else { return }
            
        // add the current gyro data sample to the data array
    }
}
```

#### 汇总传感器读数

上一步我们已经启动了加速度计和陀螺仪传感器，并设定了需要的采集频率。接下来我们需要对采集的数据进行汇总整合，以符合活动分类器的输入要求。

每当从传感器接收到新的读数后，我们将把读数添加到我们的*prediction_window*长数据数组中。

当数组达到预期大小时，应用程序就可以使用这个数组并调用模型来对新的活动进行预测了。

```swift
func addAccelerometerSampleToDataArray(accelerometerSample: CMAccelerometerData) {
    guard let dataArray = predictionWindowDataArray  else {
        return
    }
        
    dataArray[[0, currentIndexInPredictionWindow, 0] as [NSNumber]] = accelerometerSample.acceleration.x as NSNumber
    dataArray[[0, currentIndexInPredictionWindow, 1] as [NSNumber]] = accelerometerSample.acceleration.y as NSNumber
    dataArray[[0, currentIndexInPredictionWindow, 2] as [NSNumber]] = accelerometerSample.acceleration.z as NSNumber
        
    // update the index in the prediction window data array
    currentIndexInPredictionWindow += 1
        
    // If the data array is full, call the prediction method to get a new model prediction.
    // We assume here for simplicity that the Gyro data was added to the data array as well.
    if (currentIndexInPredictionWindow == ModelConstants.predictionWindowSize) {
        // predict activity
        let predictedActivity = performModelPrediction() ?? "N/A"
            
        // user the predicted activity here
            
       // start a new prediction window
        currentIndexInPredictionWindow = 0
    }
}
```

陀螺仪的数据同理，这里不再列出了。

#### 进行预测

当*prediction_window*中的读数汇总之后，就可以调用模型的预测接口来预测用户的最新活动了。

```swift
func performModelPrediction () -> String?{
     guard let dataArray = predictionWindowDataArray else { return "Error!"}
        
     // perform model prediction
     let modelPrediction = try? activityClassificationModel.prediction(features: dataArray, hiddenIn: lastHiddenOutput, cellIn: lastHiddenCellOutput)
        
     // update the state vectors
     lastHiddenOutput = modelPrediction?.hiddenOut
     lastHiddenCellOutput = modelPrediction?.cellOut
        
     // return the predicted activity -- the activity with the highest probability
     return modelPrediction?.activity
}
```


最终运行结果如下：

![](/assets/turicreate/last_result.png)

> 此结果仅仅为示例代码所示，无法保证其正确性。

## 总结

至此，关于如何使用Turi Create进行人类行为识别就可以告一段落了，但是对于一个机器学习模型的训练来说，我们这里可能有些步骤和参数的设定过于简单，因此，如果更加准确的处理数据，设定训练参数等，是个长期的探索过程。

不得不说，苹果开源的Turi Create机器学习框架，使用上简洁了很多，功能上也基本满足当下的一些机器学习任务，希望开源的Turi Create能够在社区中茁壮成长，更加完善。


## 参考资料

* [Turi Create用户指南](https://apple.github.io/turicreate/docs/userguide/)
* [Turi Create GitHub](https://github.com/apple/turicreate)
* [Core ML](https://developer.apple.com/documentation/coreml)
* [CoreMotion](https://developer.apple.com/documentation/coremotion)