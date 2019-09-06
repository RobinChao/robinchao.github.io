---
layout: post
author: Robin
title: 深入了解Core ML 3
tags: 端测计算 CoreML
categories:
- 端测计算
cover: '/assets/coreml-indepth/coreml-og.png'
---

___
在之前的文章中，介绍过[iOS 11中的机器学习](https://robinchao.github.io/2017/06/23/ios11-machine-learning.html)，简单了解了伴随iOS 11发布的Core ML框架，以及简单的使用方式等，随后，[Core ML 技术底层探秘](https://robinchao.github.io/2017/10/07/coreml-inside.html)也揭开了点Core ML背后的技术和数据结构，对Core ML相对有了一个认识。随着[Core ML vs ML Kit：哪一个移动端机器学习框架更适合你？](https://robinchao.github.io/2018/08/28/coreml-vs-mlkit.html)，简单比较了两者的差异之后，尝试了[从Keras开始构建iOS平台手写数字实时识别](https://robinchao.github.io/2019/01/22/keras-mnist-for-iOS.html)的实现，以及学习了[Apple开源机器学习框架 Turi Create 简介与实践](https://robinchao.github.io/2018/01/13/turi-create-intro.html)，并使用Turi Create进行了人类行为识别任务的尝试，[如何使用Turi Create进行人类行为识别](https://robinchao.github.io/2018/01/23/turi-create-activity-classifier.html)，对Apple的机器学习架构有了基本的认识。经过两次大版本的迭代，目前Core ML 3 也随即推出，对比之前的版本，Core ML 3 可以说已经是一个**完整的端测智能计算架构**，其中也改变了很多实现方式和支持的协议类型等，这里将再次学习，以加深对Core ML的架构认识，并`探索其端测智能计算体的使用和可能的业务`等。

___

通过[WWDC 2019 Machine Learning and Vision](https://developer.apple.com/videos/frameworks/machine-learning-and-vision)的介绍可以了解到，全新的Core ML 3 为iOS侧的机器学习增加了很多特性，其中称得上杀手锏的是**端测训练**模型的特性，以及支持更多先进的模型结构，由于增加了非常多的新的层类型，是的曾经无法执行的模型结构在端测使用也称为了可能。

![](/assets/coreml-indepth/on-device-training.png)

这次的更新，是2017年Core ML推出以来最大的一次更新。新特性的Core ML配合Apple 的 A12 神经引擎芯片，Apple可谓在端测计算能力上提升了一大步，可想而知，未来苹果会在此方向在此提高和优化，构建完善的CoreML端测计算生态系统。

这篇文章将在[Core ML 技术底层探秘](https://robinchao.github.io/2017/10/07/coreml-inside.html)的基础上，学习`mlmodel`格式的改变和新增的层类型等，而不会直接涉及`CoreML.framework`的API（事实上，除了增加了训练模型的API外，其他并没有变化）。

由于Core ML所使用的模型目前都是由其他机器学习框架训练后，进行转换后来使用的，因此如果能够详细的了解Core ML所支持的类型，那么对于设计自己的机器学习模型，并能够顺利投入到Core ML中使用，是有一定的帮助的。

## 万变不离其宗 --- proto文件

如果仅仅是查阅[CoreML.framework 的 API文档](https://developer.apple.com/documentation/coreml)，并不能找到相关的模型格式细节说明等，事实真相是，细节内容并不在API的文档中，而是在[Core ML的模型规格文档](https://apple.github.io/coremltools/coremlspecification/)里。

该规范是由许多包含protobuf消息定义的**.proto**文件组成。Protobuf是Core ML的mlmodel文件使用的序列化格式，该序列化技术是目前较为常用的，TensorFlow和ONNX也同样使用该格式。关于proto文件的具体内容，可以在[Core ML的模型规格文档](https://apple.github.io/coremltools/coremlspecification/)中找到，如果你习惯阅读源码，也可以直接查看[coremltools repo](https://github.com/apple/coremltools/tree/master/mlmodel/format)，其中有目前支持的所有内容。

在所有的proto文件中，主要的格式规格文件是**Model.proto**，该文件中定义了模型的种类，以及输入和输出的类型，另外还定义了已支持模型的不同类型等等。

在该文件中，比较重要的属性还有`Model`类中的 **specification version**，该属性决定了模型的版本以及哪些函数在mlmodel文件中支持，那个操作系统能够运行模型等。

![](/assets/coreml-indepth/spec-version.png)

> 目前官网还没有将**specification version**修改为最新的 **4**，上图是官网文件中的定义，可以预知到正式版推出后，**specification version**将是 **4**。

Core ML的模型规格版本号为 **4** 的情况下，只能运行在iOS 13 和macOS 10.15（Catalina）或更新的版本上。如果你需要运行在iOS 12或者iOS 11，需要剔除掉最新的特性。

> 当使用coremltools进行模型转换的时候，coremltools将选择最低的可能规格版本对模型格式进行兼容。v3版本的模型能够运行在iOS 12上，v2版本的模型能够运行在iOS 11.2上，v1版本能够在iOS 11上运行，当然，如果你的模型使用了任何最新的特性，就只能运行在iOS 13及以后的系统中。

## 新的模型类型 

Core ML始终支持以下模型类型（spec v1）：

* **Identity（映射）：** 仅用于测试，将输入数据传递到输出；
* **GLM（广义线性）：** 支持广义线性回归器和分类器；
* **SVM（支持向量机）：** 支持支持向量机回归和分类，底层使用的是[libsvm](https://www.csie.ntu.edu.tw/~cjlin/libsvm/)；
* **Tree ensemble（决策树模型）：** 支持回归和分类；
* **Neural networks（神经网络）：** 支持回归、分类，以及一般目标的神经网络；
* **Pipeline models（管道模型）：** 连接多个模型，形成一个机器学习工作流；
* **Feature engineering（特征工程）：** 这里指的是支持特征工程的数学模型类型，例如**One-Hot编码、缺失值处理、输入矢量化**等，这些主要用于将scikit-learn模型转换为Core ML。该模型将变为一个管道，该管道连续具有多个这些特征工程模型。


spec v2 仅仅是一个小小的更新，支持了**16-bit**浮点权重。开启之后，mlmodel文件将减小2倍，但是经过一些使用者反映，此优化并没有提高模型的运行耗时。

在Core ML 2（spec v3）中，增加了如下的模型类型：

* **Bayesian probit regressor（贝叶斯概率回归）：** 一种奇特的逻辑回归版本；
* **Non-maximum suppression（非极大值抑制）：** 对目标检测任务的后期处理有用，通常位于Pipeline的最后一层；
* **VisionFeaturePrint（可视化特征描述）：** 用于卷积神经网络，从图像中提取特征，输出规格是2048个元素的向量。也可用于图像的相似性检测等任务；
* **Create ML Support Models：** 其他来自Create ML工具的模型类型，例如文本分类、词标注等；
* **Custom models（自定义模型）：** 有时候，你可能有一个CoreML还不支持的模型类型，但是仍然希望将该模型与其他模型放在一起使用。自定义模型的功能允许开发者将模型参数和数据放置在mlmodel文件中，同时将自定义的逻辑放在应用程序中。

spec v3 还增加了以减小mlmodel文件大小的**权重量化**功能以及灵活的输入大小，API层面增加了批处理预测，并为顺序数据提供了更好的支持。

在Core ML 3（spec v4）中，增加了如下的模型类型：

* **k-Nearest Neighbors（k-NN，k近邻）：** k-NN分类器；
* **ItemSimilarityRecommender（基于相似度的推荐器）：** 可以使用该类型构建个性化推荐模型
* **SoundAnalysisPreprocessing（声音分析预处理）：** 支持Create ML的声音分类模型。输入音频信号样本，并会转换为mel频谱图，可以在Pipeline中用做音频特征模型的输入；
* **Gazetteer：** 支持Create ML的`MLGazetteer`模型，使用自然语言处理框架中的`NLTagger`。一个gazetteer是一个用于单词和短语的花式查找表；
* **WordEmbedding：** 支持Create ML的新`MLWordEmbedding`模型，该模型是单词及其嵌入向量的字典。也用于自然语言框架；
* **Linked models：** 对应用程序包中另一个mlmodel文件（编译版本，mlmodelc）的引用。这使得可以跨多个分类器重用昂贵的特征提取器 - 如果两个不同的管道使用相同的链接模型，则只会加载一次。

目前，`Model`对象的属性中增加了**isUpdatable**属性，当该属性是`true`时，代表模型可以在端测进行训练，目前仅支持神经网络和k-NN模型。

**k-NN模型**是一个相对简单的机器学习模型，非常适合在端测进行训练，常见的方法是使用固定的神经网络，例如VisionFeaturePrint，从输入数据中提取特征，然后使用k-NN对这些特征向量进行分类。这样的模型在训练时很快，因为它只是记住了你输入的样本，并没有做任何实际的学习。

**k-NN模型**的一个缺点是，当记忆了非常多的样本时，预测就会变得很慢，但是在Core ML中，使用了一个k-NN的变种**[K-D树](https://zh.wikipedia.org/wiki/K-d%E6%A0%91)**。(下图是一个三维k-d树)

![](/assets/coreml-indepth/3dtree.png)

## 神经网络的更新

在Core ML 2的版本中，仅支持40中不同的神经网络层类型，Core ML 3中增加了超过100中层类型。但是并不是所有的都是新增，部分是对旧的层类型的改进，以支持或者更加适合处理柔性张量的处理等。

在Core ML 2及之前的版本，流入神经网络的数据总是一个5等级的张量，也就是说每个张量都是由如下5个维度组成的：

```sh
(sequence length, batch size, channels, height, width)
```

当神经网络的输入是图像的话，这个数据结构非常适合，但是对于其他数据结构就不那么适合了。

例如，在处理一维向量的神经网络中，应该使用`channels`来描述向量的大小，并将其他维度设置为1，在这种情况下，输入的张量形状是`(1, batch size, number of elements, 1, 1)`。虽然这样的方式也可以解决问题，但是对于开发者来说，无意义的参数设置就是浪费时间，因此在Core ML 3中新增了很多新层来支持任意等级和形状的张量，使Core ML对于处理图像之外的数据也更适合。

> 上面部分的描述均来自proto文件中的描述和解释的理解，在其他的文档中并不会去解释和说明这些内容，因此有兴趣的话，可以详细阅读下proto文件的内容，以加深理解。

在[NeuralNetwork.proto](https://github.com/apple/coremltools/blob/master/mlmodel/format/NeuralNetwork.proto)文件中，接近6000行的内容，对Core ML中神经网络的内容进行了描述和定义。

其中，主对象是**NeuralNetwork**，还有另外两个变种**NeuralNetworkClassifier**和**NeuralNetworkRegressor**，不同的是，普通的神经网络输出的是MultiArray对象或者image，分类器输出的是包含类别和对应预测概率的字典，回归器输出的是一个数值。除了输出和响应的解释不同，但是这三种模型的工作方式是相同的。

**NeuralNetwork**对象含有一个层的列表，以及任何图像输入的预处理选项列表。Core ML 3 增加了一些描述的新特性：

* 输入的MultiArray类型是如何转换为张量的。此时，开发者可以选择老的方式，创建一个秩为5的张量，或者使用新的方式。大多数情况下的输入类型都不是图像，因此这里将是常用的方法。
* 输入的Image类型是如何转换为张量的。更换了老的秩为5的张量，开发者可以使用秩为4的张量，`(batch size, channels, height, width)`。这里去除了在图像处理中不需要的`sequence length`维度。
* 训练模型的超参数。在**NetworkUpdateParameters**中描述。


关于**端测训练**的能力，Core ML 3中也相应增加了一些支持的描述，具体如下：

* **isUpdatable**位于**Model**对象中，表示模型是否可以被训练或再训练；
* 在任何希望进行训练的层中，**isUpdatable**必须设置为**true**。使用该参数，开发者可以控制某些层的训练与否。目前，端测训练仅支持卷积层和全链接层；
* **WeightParams**是训练时可学习的参数，只有在**isUpdatable**为**true**的时候会起效；
* 在训练前，需要为模型定义一个训练输入，该输入将用于训练中损失函数的实际标签。

另外，在**NetworkUpdateParameters**对象中，定义了模型训练的一些方法和参数等，具体描述如下：

* **lossLayers**：指定使用那种损失函数，目前支持分类交叉熵和MSE（均方误差）。在mlmodel文件中，损失函数是另外的一层，含有两个属性，模型输出层名称和对应目标类别的训练输入名称。对于交叉熵损失函数，输入必须连接softmax输出层；
* **optimizer**：目前支持SGD（随机梯度下降）和Adam（可替代SGD的一种一阶优化算法）；
* **epochs**：模型训练的迭代次数。
* **shuffle**：每次迭代时，数据是否需要随机重新排序；
* **seed**：随机种子参数。

## Core ML 2 中的神经网络层
 

 对于神经网络来说，最有意思的还算各种神经网络层了，在Core ML第一个版本中，支持一下的神经网络层类型：

* **Convolution（卷积层）：**仅支持2维度，但是可以设置内核宽度和高度为1来使用1维。同时支持空洞卷积或扩张卷积、分组卷积和反卷积；
* **Pooling（池化层）：**支持max、average、L2，以及全局Pooling；
* **Fully-connected（全链接层）：**也被称为内积层或密集层；
* **Activation functions（激活函数）：**支持linear、ReLU、leaky ReLU、thresholded ReLU、PReLU、tanh、scaled tanh、sigmoid、hard sigmoid、ELU、softsign、softplus、parametric soft plus；
* **Batch normalization（批量标准化层）：**
* **Normalization（标准化层）：**mean & variance, L2 norm, and local response normalization (LRN)；
* **Softmax：**在**NeuralNetworkClassifier**中的最后一层使用；
* **Padding（填充）：**用于在图像张量的边缘周围添加额外的零填充。卷积和池化层已经可以自己处理填充，但是使用此层可以执行诸如反射或复制填充之类的操作；
* **Cropping（剪裁）：**用于去除张量边缘周围的像素；
* **Upsampling（上采样）：**最近邻或双线性上采样整数比例因子；
* **Unary operations（一元处理）：**sqrt, 1/sqrt, 1/x, x^power, exp, log, abs, thresholding；
* **Tensors Element-wise operations（两个及以上张量元素操作）：**add, multiply, average, maximum, minimum；
* **Tensor Element-wise operations（两单个张量元素操作）：**乘以比例因子，增加偏差；
* **Reduction operations（降维）：** sum, sum of natural logarithm, sum of squares, average, product, L1 norm, L2 norm, maximum, minimum, argmax；
* **Dot product（张量点积）：**计算余弦相似度；
* **Reorganize（张量重组）：** reshape, flatten, permute, space-to-depth, depth-to-space；
* **Concat, split, and slice（张量联合、分割、切片）：**张量合并或扩张；
* **Recurrent neural network layers（递归神经网络层）：**basic RNN, uni- and bi-directional LSTM, GRU (unidirectional only)；
* **Sequence repeat（序列复制）：**多次复制给定的输入序列；
* **Embeddings**
* **Load constant（负载常数）：**可以用于向一些其他层提供数据，例如对象检测模型中的锚定区。


Apple在 Spec v2 中增加了对神经网络中自定义层的支持。这个补充，对更多模型的转换有了很大的帮助。

在mlmodel文件中，自定义图层只是一个占位符，可能具有经过训练的权重和配置参数。在应用程序中，应该提供该层功能的Swift或Objective-C实现，并且可能还有一个Metal版本以及在GPU上运行它。 不幸的是，神经引擎目前不是自定义图层的选项，该功能也仅支持传统机器学习模型。

例如，如果模型需要不在上面列表中的激活函数，则可以将其实现为自定义层。也可以通过巧妙地组合其他一些图层类型来完成此操作。例如，可以通过进行常规ReLU，然后将数据乘以-1，将阈值乘以-6，最后再乘以-1来制作ReLU6。这需要4个不同的层，但理论上，Core ML框架可以在运行时优化它。

在Core ML 2（Spec v3）中，添加了以下图层类型：

* **Resize bilinear：**与上采样层（仅接受整数比例因子）不同，这使您可以将双线性调整为任意图像大小；
* **Crop-resize：**用于从张量中提取感兴趣的区域。这可以用于实现掩模R-CNN中使用的RoI Align层。

在Core ML 3（Spec v4）放宽了对这些现有层类型的要求，除了添加了一大堆新层之外，Core ML 3还使现有的图层类型更加灵活。


## 新的神经网络层

上文提到，此次更新，新增了100多个层，下面会一一查看都有哪些层，更加详细的内容可参考[NeuralNetwork.proto](https://github.com/apple/coremltools/blob/master/mlmodel/format/NeuralNetwork.proto)描述文件。

Core ML 3 为元素明确的一元操作添加以下层：

* **ClipLayer**：夹在最大值和最小值；
* **CeilLayer、FloorLayer**：对张量进行ceil和floor运算；
* **SignLayer**：告知数字是正数，零数还是负数；
* **RoundLayer**：将张量的值四舍五入为整数；
* **Exp2Layer**：对张量元素进行2^x运算；
* **SinLayer, CosLayer, TanLayer, AsinLayer, AcosLayer, AtanLayer, SinhLayer, CoshLayer, TanhLayer, AsinhLayer, AcoshLayer, AtanhLayer**：（双曲线）三角函数；
* **ErfLayer**：计算高斯误差函数。

这次和运算相关的增加，扩展了Core ML支持的数学原语的数量，与已有的数学函数不同，它们可以处理任何等级的张量。

这里只有一个新的激活函数：

* **GeluLayer**：高斯误差线性单元激活函数，精确的或使用tanh或S形近似。

当然，也可以使用任何一元函数作为激活函数，或通过组合不同的数学层来创建一个。

另外还增加了用于比较张量的新的层类型：

* **EqualLayer, NotEqualLayer, LessThanLayer, LessEqualLayer, GreaterThanLayer, GreaterEqualLayer**
* **LogicalOrLayer, LogicalXorLayer, LogicalNotLayer, LogicalAndLayer**

当条件为真时，这些函数输出一个新的张量，其值为1，反之为0。这些图层类型支持广播，因此您可以比较不同等级的张量。您还可以将张量与（硬编码）标量值进行比较。

这些图层类型有用的一个地方是使用新的控制流操作（见下文），以便您可以根据比较的结果进行分支，或者创建一个循环，该循环一直重复直到某个条件变为false。

以前，在两个或更多张量之间有一些用于元素操作的图层。Core ML 3添加了一些新类型，从名称可以看出这些新增的更加灵活，因为它们完全支持NumPy风格：

* **AddBroadcastableLayer**： 加法
* **SubtractBroadcastableLayer**：减法
* **MultiplyBroadcastableLayer**：乘法
* **DivideBroadcastableLayer**：除法
* **FloorDivBroadcastableLayer**：除法返回四舍五入的整数结果
* **ModBroadcastableLayer**：除法余数
* **PowBroadcastableLayer**：幂运算
* **MinBroadcastableLayer, MaxBroadcastableLayer**：最大、最小

在新的Core ML中，内置了大量的张量操作方法，而且不仅是图像，也可以通过一个或者多个维度的变换来进行张量缩小等。

* **ReduceSumLayer**：计算指定维度的总和
* **ReduceSumSquareLayer**：计算张量元素的平方和 
* **ReduceLogSumLayer**：计算元素的自然对数之和
* **ReduceLogSumExpLayer**：指定元素进行加和，再取自然对数
* **ReduceMeanLayer**：计算元素的平均值
* **ReduceProdLayer**：所有元素的乘积
* **ReduceL1Layer, ReduceL2Layer**： L1、L2标准化
* **ReduceMaxLayer, ReduceMinLayer**：寻找最大值、最小值
* **ArgMaxLayer, ArgMinLayer**：最大值、最小值的索引
* **TopKLayer**：找到k个顶部（或底部）值及其索引。

关于数学的计算，Core ML 3还增加了如下的类型：

* **BatchedMatMulLayer**：两个输入张量上的通用矩阵乘法，或单个输入张量和一组固定的权重（加上可选的偏差）。支持广播并可在进行乘法之前转置输入；
* **LayerNormalizationLayerParams**：一个简单的归一化层，它减去β（例如均值）并除以γ（例如标准偏差），两者都作为固定权重提供。这与现有的MeanVarianceNormalizeLayer不同，后者执行相同的公式但实际上在推理时计算张量的均值和方差。

许多其他现有操作已经扩展到使用任意大小的张量，也称为秩-N张量或N维张量。您可以通过名称中的“ND”识别此类图层类型：

* **SoftmaxNDLayer**
* **ConcatNDLayer**
* **SplitNDLayer**
* **TransposeLayerParams**
* **EmbeddingNDLayer**
* **LoadConstantNDLayerParams**

Core ML 3为我们提供了两个新的切片层，支持在任何轴上切片：

* **SliceStaticLayer**
* **SliceDynamicLayer**

静态基本上意味着“预先知道此操作的一切”，而动态意味着“此操作的参数可以在运行之间改变”。例如，图层的静态版本可能具有硬编码的outputShape属性，而动态版本每次都可以使用不同的输出形状。

因为Core ML不再局限于基于静态图像的模型，而是现在还包含控制流和其他动态操作的方法，因此它必须能够以各种奇特的方式操纵张量。

* **GetShapeLayer**
* **BroadcastToStaticLayer, BroadcastToLikeLayer，BroadcastToDynamicLayer**
* **RangeStaticLayer, RangeDynamicLayer**
* **FillStaticLayer, FillLikeLayer, FillDynamicLayer**

其中一些图层类型有三种不同的变体：Like，Static和Dynamic。

* **Static**：该图层的所有属性都在mlmodel文件中进行了硬编码。
* **Like**：需要一个额外的输入张量并输出一个与输入具有相同形状的新张量。
* **Dynamic**：它还需要一个额外的输入张量，但这次它不是那个重要的形状，而是它的内容。

Core ML 3还允许您通过随机分布采样创建新的张量：

* **RandomNormalStaticLayer, ...LikeLayer, ...DynamicLayer**
* **RandomUniformStaticLayer, ...LikeLayer, ...DynamicLayer**
* **RandomBernoulliStaticLayer, ...LikeLayer, ...DynamicLayer**
* **CategoricalDistributionLayer**

另外的一些变体层：

* **SqueezeLayer**：删除任何大小为1的维度
* **ExpandDimsLayer**：和**SqueezeLayer**相反
* **FlattenTo2DLayer**：将输入张量展平为二维矩阵
* **ReshapeStaticLayer, ReshapeLikeLayer, ReshapeDynamicLayer**
* **RankPreservingReshapeLayer**：类似NumPy中的**reshape(..., -1)**

除了任意张量的连续和分割操作外，Core ML 3还增加了以下张量操作操作：

* **TileLayer**：重复张量一定次数
* **StackLayer**：沿着新轴连接张量
* **ReverseLayer**：反转输入张量的一个或多个维度
* **ReverseSeqLayer**：对于存储数据序列的张量，反转序列
* **SlidingWindowsLayer**：在输入数据上滑动一个窗口，并在每一步返回一个带有窗口内容的新张量

同样支持聚集和缩放：

* **GatherLayer, GatherNDLayer, GatherAlongAxisLayer**：给定一组索引，只保留输入张量的那些索引
* **ScatterLayer, ScatterNDLayer, ScatterAlongAxisLayer**：将一个张量的值复制到另一个张量中，但仅限于给定的索引。除了复制之外，还有其他累积模式：加，减，乘，除，最大和最小。

除了能够选择层之外，还可以选择某些指定的层：

* **WhereNonZeroLayer**：创建一个只有非零元素的新张量。您可以将此与张量比较中的掩码张量一起使用，例如LessThanLayer。
* **WhereBroadcastableLayer**：采用三个输入张量，两个数据张量和一个包含（真）或零（假）的掩码。返回包含第一个数据张量或第二个数据张量元素的新张量，具体取决于掩码中的值是true还是false。
* **UpperTriangularLayer, LowerTriangularLayer**：将对角线下方或上方的元素归零
* **MatrixBandPartLayer**：将中心带外的元素归零

目前，coremltools 3.0的Beta中还隐藏了一些新的图层类型：

* **ConstantPaddingLayer**：在张量周围添加一定量的填充。与现有的填充图层不同，此图层适用于任何轴，而不仅仅是宽度和高度尺寸。
* **NonMaximumSuppressionLayer**：已经有一个单独的模型类型用于在边界框上进行NMS，您可以在对象检测检测模型之后将其放入管道中，但现在也可以直接在神经网络内部进行NMS。

在Core ML 3中还增加了控制流层，这些层是激动人心的，极大的提高了神经网络的结构适配广度：

* **BranchLayer**
* **LoopLayer**
* **LoopBreakLayer**
* **LoopContinueLayer**
* **CopyLayer**

这些控制流相关的层，coremltools给出了使用样例，详细使用方式可以参考[Neural_network_control_flow_power_iteration.ipynb](https://github.com/apple/coremltools/blob/master/examples/Neural_network_control_flow_power_iteration.ipynb)。

至此，Core ML 3中的新的内容基本罗列出来了，可以看到，在新的升级更新中，大多数都是针对层张量的创建、整形和操作，还有很多的数学运算能力的提升，但是通过这些操作，开发者已经可以通过定制、组合等来支持新的层类型，然后实现不同的AI任务等。Apple在一步一步地强化着Core ML的体系，以增强端测AI的能力，

## 参考资料

* [coremltools](https://github.com/apple/coremltools)
* [Core ML Documentation](https://developer.apple.com/documentation/coreml)
* [An in-depth look at Core ML 3](https://machinethink.net/blog/new-in-coreml3/)