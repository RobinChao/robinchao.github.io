---
layout: post
author: Robin
title: SVM学习 --- 基于智能手机传感器数据的人类行为识别（二）
tags: 机器学习 行为识别 SVM
categories:
  - 机器学习
  - SVM
---


> 本文已发布到infoQ，点击链接可查看。[infoQ](http://www.infoq.com/cn/articles/human-behavior-recognition-based-on-smart-phone-sensor-data)

在[基于智能手机传感器数据的人类行为识别（一）](https://robinchao.github.io/blog/2017/02/human-activity-recognize-1)中，我们简单介绍了支持向量机以及如何使用LIBSVM类库和加速度传感器数据进行特征的抽取、模型的训练、参数的调优和模型的测试等，在本文中，将使用上篇最终得到的模型文件，以及LIBSVM类库，在iOS平台下构建一个能够识别当前客户端用户的行为类型的应用。

随着移动终端设备的性能越来越高，其集成的传感器设备也越来越多，侦测精度越来越高的情况，应用于移动终端设备上的机器学习应用也多了起来。在iOS平台下，苹果官方的很多应用中也呈现出了机器学习的影子。例如iOS 10 系统中的相册，能够进行人脸识别并进行照片的自动分类、邮件中的自动垃圾邮件归类、Siri智能助理、健康应用中的用户运动类型分类等。

### 用户的运动类型

在iOS系统的健康应用中，可以看到你的运行类型，其中包含了行走、跑步、爬楼梯、步数、骑自行车等类型。
 
<img src="/assets/har/health.png" width="600" height="400"/> 

在iOS SDK中也提供了一个专用于运动类型获取的类**[CMMotionActivityManager](https://developer.apple.com/reference/coremotion/cmmotionactivitymanager)**，其中包含了

``` objc
stationary 
walking 
running 
automotive 
cycling 
unknown 
```
几种行为类型，但是在使用的过程中，可能会遇到当前行为和此类给出的结果不相同或者同一时刻有东中类型的情况，这里引用苹果给出的一段结论：

> An estimate of the user's activity based on the motion of the device.
	The activity is exposed as a set of properties, the properties are not
	mutually exclusive.
 
> For example, if you're in a car stopped at a stop sign the state might
look like:
stationary = YES, walking = NO, running = NO, automotive = YES

> Or a moving vehicle,
stationary = NO, walking = NO, running = NO, automotive = YES

> Or the device could be in motion but not walking or in a vehicle.
stationary = NO, walking = NO, running = NO, automotive = NO.
Note in this case all of the properties are NO.

因此用户行为的识别并不是严格意义上的准确的，在机器学习领域，预测都会有一个概率的输出，引申出的就是正确率，**正确率**也是评估一个机器学习模型的标准之一。

### 关于加速度传感器

苹果的移动设备中，集成了[多种传感器](https://developer.apple.com/reference/coremotion)，本文所演示的仅仅使用加速度传感器，你也可以增加传感器类型，提高数据的维度等。

<img src="/assets/har/cmdm-axes.png" width="400" height="400"/> 

加速度传感器数据 [CMAccelerometerData](https://developer.apple.com/reference/coremotion/cmaccelerometerdata) 的类型为CMAcceleration，提供了三轴加速度值，如下： 

``` objc
typedef struct {
	double x;
	double y;
	double z;
} CMAcceleration;
// A structure containing 3-axis acceleration data.
```
此加速度值是当前设备总的加速度值，想要获取加速度分量的时候，可以使用[CMDeviceMotion](https://developer.apple.com/reference/coremotion/cmdevicemotion)进行获取。

### 构建iOS项目，收集传感器数据

在上篇中，我们已经知道，LIBSVM具有多种语言的接口，这里我们直接使用其C语言接口，在iOS项目中构建SVM分类器。

#### 1. 传感器数据收集

首先需要收集加速度传感器数据，并进行数据特征抽取和数据准备，以便SVM算法识别使用。在iOS的[CoreMotion](https://developer.apple.com/reference/coremotion)框架中，已经提供了获取加速度传感器的API，开发者可以直接调用接口获取加速度传感器数据：

``` objc
 	CMMotionManager *motionManager = [[CMMotionManager alloc] init];
    if ([motionManager isAccelerometerAvailable]) {
        [motionManager setAccelerometerUpdateInterval:0.02];
        
        startTime = [[NSDate date] timeIntervalSince1970];
        [motionManager startAccelerometerUpdatesToQueue:[NSOperationQueue mainQueue] withHandler:^(CMAccelerometerData * _Nullable accelerometerData, NSError * _Nullable error) {
            if (error) {
                NSLog(@"%@", error.description);
            }else{
                [self handleDeviceAcc:accelerometerData];
            }
        }];
    }
```

#### 2. 数据批量化处理

我们在本文开始介绍训练数据集的时候，提到了数据的采集频率是 **20 Hz**，因此我们在进行数据采集的时候也需要同样的频率，并且将传感器数据进行批量化处理，以便于模型识别时具有合适数量的数据。

``` objc
	NSArray *valueArr = @[
            @(accelerometerData.acceleration.x * g_value),
            @(accelerometerData.acceleration.y * g_value),
            @(accelerometerData.acceleration.z * -g_value)];
    
    NSMutableDictionary *sample = [NSMutableDictionary dictionary];
    [sample setValue:currenStatus forKey:@"status"];
    [sample setValue:@"acc" forKey:@"sensorName"];
    [sample setValue:@([self getTimeStampByMiliSeconds]) forKey:@"timestamp"];
    [sample setValue:valueArr forKey:@"values"];
    
    if (sampleDatas == nil) {
        sampleDatas = [NSMutableArray array];
    }
    
    if ([sampleDatas count] == 256) {
        NSArray *readySamples = [NSArray arrayWithArray:sampleDatas];
        sampleDatas = nil;
        [self stopMotionAccelerometer];
        
        [self recognitionData:[readySamples copy]];
    }else{
        [sampleDatas addObject:sample];
    }

```

#### 3. 特征抽取

在开始此步骤之前，我们需要导入LIBSVM的类库到项目工程中，这里仅需要导入**svm.h**和**svm.cpp**两个文件即可。

在训练模型的时候，我们使用了五种特征，最终生成所需要的数据格式，这里同样，我们也需要针对数据进行特征提取，并重新组合数据成为LIBSVM所要求的数据格式：

``` objc
	for (NSUInteger index = 0; index < [raw_datas count]; index++) {
        NSDictionary *jsonObject = raw_datas[index];
        NSArray *valuesArray = jsonObject[@"values"];
        if (!valuesArray || valuesArray.count <= 0) {
            break;
        }
        id acc_x_num = valuesArray[0];
        id acc_y_num = valuesArray[1];
        id acc_z_num = valuesArray[2];

        acc_x_axis[index] = acc_x_num;
        acc_y_axis[index] = acc_y_num;
        acc_z_axis[index] = acc_z_num;

        gravity[index] = @(sqrt(pow([acc_x_num doubleValue], 2) + pow([acc_y_num doubleValue], 2) + pow([acc_z_num doubleValue], 2)));
 
    }

    NSMutableArray *values = [NSMutableArray array];
    /* mean Feature */{
        struct svm_node node_x_mean = {0, [StatisticFeature mean:acc_x_axis]};
        NSValue *node_x_mean_value = [NSValue valueWithBytes:&node_x_mean objCType:@encode(struct svm_node)];
        [values addObject:node_x_mean_value];

        struct svm_node node_y_mean = {1, [StatisticFeature mean:acc_y_axis]};
        NSValue *node_y_mean_value = [NSValue valueWithBytes:&node_y_mean objCType:@encode(struct svm_node)];
        [values addObject:node_y_mean_value];

        struct svm_node node_z_mean = {2, [StatisticFeature mean:acc_z_axis]};
        NSValue *node_z_mean_value = [NSValue valueWithBytes:&node_z_mean objCType:@encode(struct svm_node)];
        [values addObject:node_z_mean_value];

        struct svm_node node0 = {3, [StatisticFeature mean:gravity]};
        NSValue *value0 = [NSValue valueWithBytes:&node0 objCType:@encode(struct svm_node)];
        [values addObject:value0];
    }
    /* max Feature */{
        struct svm_node node_x_max = {4, [StatisticFeature max:acc_x_axis]};
        NSValue *node_x_max_value = [NSValue valueWithBytes:&node_x_max objCType:@encode(struct svm_node)];
        [values addObject:node_x_max_value];

        struct svm_node node_y_max = {5, [StatisticFeature max:acc_y_axis]};
        NSValue *node_y_max_value = [NSValue valueWithBytes:&node_y_max objCType:@encode(struct svm_node)];
        [values addObject:node_y_max_value];

        struct svm_node node_z_max = {6, [StatisticFeature max:acc_z_axis]};
        NSValue *node_z_max_value = [NSValue valueWithBytes:&node_z_max objCType:@encode(struct svm_node)];
        [values addObject:node_z_max_value];

        struct svm_node node1 = {7, [StatisticFeature max:gravity]};
        NSValue *value1 = [NSValue valueWithBytes:&node1 objCType:@encode(struct svm_node)];
        [values addObject:value1];
    }
    /* min Feature */{
        struct svm_node node_x_min = {8, [StatisticFeature min:acc_x_axis]};
        NSValue *node_x_min_value = [NSValue valueWithBytes:&node_x_min objCType:@encode(struct svm_node)];
        [values addObject:node_x_min_value];

        struct svm_node node_y_min = {9, [StatisticFeature min:acc_y_axis]};
        NSValue *node_y_min_value = [NSValue valueWithBytes:&node_y_min objCType:@encode(struct svm_node)];
        [values addObject:node_y_min_value];

        struct svm_node node_z_min = {10, [StatisticFeature min:acc_z_axis]};
        NSValue *node_z_min_value = [NSValue valueWithBytes:&node_z_min objCType:@encode(struct svm_node)];
        [values addObject:node_z_min_value];

        struct svm_node node2 = {11, [StatisticFeature min:gravity]};
        NSValue *value2 = [NSValue valueWithBytes:&node2 objCType:@encode(struct svm_node)];
        [values addObject:value2];
    }
    /* stev Feature */{
        struct svm_node node_x_stev = {12, [StatisticFeature stev:acc_x_axis]};
        NSValue *node_x_stev_value = [NSValue valueWithBytes:&node_x_stev objCType:@encode(struct svm_node)];
        [values addObject:node_x_stev_value];

        struct svm_node node_y_stev = {13, [StatisticFeature stev:acc_y_axis]};
        NSValue *node_y_stev_value = [NSValue valueWithBytes:&node_y_stev objCType:@encode(struct svm_node)];
        [values addObject:node_y_stev_value];

        struct svm_node node_z_stev = {14, [StatisticFeature stev:acc_z_axis]};
        NSValue *node_z_stev_value = [NSValue valueWithBytes:&node_z_stev objCType:@encode(struct svm_node)];
        [values addObject:node_z_stev_value];

        struct svm_node node3 = {15, [StatisticFeature stev:gravity]};
        NSValue *value3 = [NSValue valueWithBytes:&node3 objCType:@encode(struct svm_node)];
        [values addObject:value3];
    }
```

这里需要注意的是，特征的顺序必须和模型训练时训练数据集中的特征顺序一致，否则预测的结果将出现严重的偏差。

#### 4. 导入模型文件并加载

完成了数据准备之后，我们导入之前训练好的模型文件**raw_bestP_trained.model**到项目中，然后使用LIBSVM提供的模型加载方法，加载模型到**svm_model**结构体对象：

``` objc
	struct svm_model * model = svm_load_model([model_dir UTF8String]);

    if (model == NULL) {
        NSLog(@"Can't open model file: %@",model_dir);
        return nil;
    }
    if (svm_check_probability_model(model) == 0) {
        NSLog(@"Model does not support probabiliy estimates");
        return nil;
    }
```

#### 4. 行为识别

LIBSVM提供了多个方法进行预测，为了最终看到预测的概率，我们使用

``` c
double svm_predict_probability(const struct svm_model *model, 
									const struct svm_node *x, 
									double* prob_estimates);
```

方法，在输出预测结果的时候，会带有对应的概率：

``` objc
	//Type of svm model
    int svm_type = svm_get_svm_type(model);
    //Count of labels
    int nr_class = svm_get_nr_class(model);
    //Label of svm model
    int *labels = (int *) malloc(nr_class*sizeof(int));
    svm_get_labels(model, labels);
    
    // Probability of each possible label in result
    double *prob_estimates = (double *) malloc(nr_class*sizeof(double));
    // Predicting
    // result of prediction including:
    // - Most possible label
    // - Probability of each possible label
    double label = 0.0;
    if (svm_type == C_SVC || svm_type == NU_SVC) {
          label = svm_predict_probability(model, X, prob_estimates); 
          NSLog(@"svm_predict_probability label: %f",label);
    }else{
        NSLog(@"svm_type is not support !!!");
        return nil;
    }

```

通过以上的预测之后，最终的预测结果就是**label**，并在会在**prob_estimates**中输出各个分类标签的预测概率。

**！！注意 ！！**
** prob_estimates 中仅仅会输出概率，并不会输出概率和标签的对应关系。prob_estimates中的概率顺序是和模型中的输入标签顺序一致的，需要注意！ **

最终的预测结果如下：

``` shell
label: 4 -- prob: 0.491513 
label: 1 -- prob: 0.285421 
label: 2 -- prob: 0.119973 
label: 3 -- prob: 0.096848 
label: 0 -- prob: 0.002580 
label: 5 -- prob: 0.003665
```


### 关于模型的评估

分类模型的度量有很多方式，例如混淆矩阵（Confusion Matrix）、ROC曲线、AUC面积、Lift（提升）和Gain（增益）、K-S图、基尼系数等，这里我们使用ROC曲线评估我们最终得到的模型，以查看模型的质量，最终的ROC曲线图如下：

![](/assets/har/roc.png) 

可以看到该模型针对某些行为的识别能力较好，例如站立、慢跑，但是对另一些行为的识别却不怎么好了，例如下楼梯。

### 总结

可以看到SVM在分类问题上能够很好的识别特征进行类别区分。由于篇幅原因，本文中并没有对数据的特征进行更加细致的选择和抽取，可能会导致一些行为类型的识别不能达到理想的效果，但是相信在足量的数据下，进行更加细致的特征工程后，利用SVM在分类能力上的优势，能够构建出更加优秀的人类行为类型识别的智能应用。