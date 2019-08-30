---
layout: post
author: Robin
title: 从Keras开始构建iOS平台手写数字实时识别
tags: 机器学习 Keras MNIST
categories:
- 机器学习
cover: '/assets/keras-mnist-for-ios/cover.jpg'
---

本文将介绍如何构建和训练一个深度学习网络来识别手写数字，以及如何将训练所得的深度网络模型转换为iOS平台的机器学习框架CoreML格式，并集成进iOS应用程序中以实时识别数字等。

# **10步之内完成模型的构建、训练和发布使用**

**TLDR；**

本文中暂时不会介绍卷积神经网络的细节内容，例如如何使用卷积层、池化层训练深度学习网络，以及如何使用预训练模型识别目标等，相关卷积神经网络细节的内容将会放在本文内容之后，进行详细的介绍。本文旨在介绍如何一步一步的从数据的获取、整理、模型的构建、训练以及后面的格式转换、使用等介绍Keras框架的基本使用和如何使用CoreML体系在一个实实在在的应用程序中使用模型等。

下图是最终结果的预览：

![](https://liip.rokka.io/www_inarticle/812493/output.gif)

接下来，开始一步步的实现相关的过程等。

## 1. **如何开始**

To have a fully working example I thought we’d start with a toy dataset like the [MNIST set of handwritten letters](https://en.wikipedia.org/wiki/MNIST_database) and train a deep learning network to recognize those. Once it’s working nicely on our PC, we will port it to an iPhone X using the [CoreML standard](https://developer.apple.com/documentation/coreml).

在计算机程序设计学习的过程中，几乎都是以一个经典的“Hello World”程序开始的。而在机器学习领域，同样具有类似“Hello World”的一个经典入门级数据集——[MNIST](https://en.wikipedia.org/wiki/MNIST_database)，该数据集是一系列手写数字0到9的图片文件，这里的目的是使用这个数据集训练一个深度学习网络来识别它们。在开始之前，你或许对iOS平台的CoreML以及keras还很陌生，你可以先了解一下它们的体系和设计：

- CoreML：

[Core ML | Apple Developer Documentation](https://developer.apple.com/documentation/coreml)

- Keras：

[Keras 中文文档](https://keras.io/zh/)

## 2. 获取数据

在大多数的Python机器学习类库中，都有内置的数据集访问接口，以方便使用者的使用，在Keras中也不例外，可以很方便的使用其内置的数据集访问接口获取数据集，具体的接口定义在`keras.datasets`中，具体的使用如下：

```python
# 使用Keras内置数据集访问接口导入数据集并对数据集进行转换
    from keras.datasets import mnist
    from keras.utils import np_utils
    from keras import backend as K
    
    def mnist_data():
        # 定义输入图像的维度
        img_rows, img_cols = 28, 28
        # 加载数据集
        (X_train, Y_train), (X_test, Y_test) = mnist.load_data()
    
        if K.image_data_format() == 'channels_first':
            X_train = X_train.reshape(X_train.shape[0], 1, img_rows, img_cols)
            X_test = X_test.reshape(X_test.shape[0], 1, img_rows, img_cols)
            input_shape = (1, img_rows, img_cols)
        else:
            X_train = X_train.reshape(X_train.shape[0], img_rows, img_cols, 1)
            X_test = X_test.reshape(X_test.shape[0], img_rows, img_cols, 1)
            input_shape = (img_rows, img_cols, 1)
    
        # 数据缩放，将原来的 [0, 255] 缩放至 [0, 1]
        X_train = X_train.astype('float32')/255
        X_test = X_test.astype('float32')/255
    
        # 对原始数据中的目标值进行One-Hot Encoding，使得目标数据更加的稀疏
        Y_train = np_utils.to_categorical(Y_train, 10)
        Y_test = np_utils.to_categorical(Y_test, 10)
    
        # 返回结果
        return (X_train, Y_train), (X_test, Y_test), input_shape
    
    (X_train, Y_train), (X_test, Y_test), input_shape = mnist_data()
```

## 3. 正确地编码

当处理图片数据的时候，必须要区分想要的编码方式。Keras是一个可以处理多个“后端”的高级库，例如[Tensorflow](https://www.tensorflow.org/), [Theano](http://deeplearning.net/software/theano/) 和 [CNTK](https://www.microsoft.com/en-us/cognitive-toolkit/)，首先我们要了解我们所使用的后端是如何编码数据的。在Keras默认使用的TensorFlow后端中，针对图像的处理通常是以“通道优先”或“通道末尾”的方式进行编码的，因此在我们的使用TensorFlow作为后端的时候，编码结果其实是一个张量，其形状为(batch_size, rows, cols, channels)。意味着首先是输入的batch_size，然后输入28行28列的图像维度，最后输入1作为通道数，因为我们使用的是灰度图像数据。

我们可以看看前6张图像具体是什么样子，可以使用如下代码查看：

```python
# 可视化数据集中前6张图像
    import matplotlib.pyplot as plt
    %matplotlib inline
    import matplotlib.cm as cm
    import numpy as np
    
    (X_train, y_train), (X_test, y_test) = mnist.load_data()
    
    fig = plt.figure(figsize=(20,20))
    for i in range(6):
        ax = fig.add_subplot(1, 6, i+1, xticks=[], yticks=[])
        ax.imshow(X_train[i], cmap='gray')
        ax.set_title(str(y_train[i]))
    plt.show()
```

![](https://liip.rokka.io/www_inarticle/7cce04/numbers.png)

## 

## 4. 规范化数据

可以看到，在黑色背景中显示了白色数字，每一张图像中的数字都是居中的，而且分辨率都很低——在这个例子中我们使用的是28x28像素。

你可能已经注意到，在上述获取数据的部分，我们对每一张图片除以255来缩放了图像像素，这导致像素值在0和1之间，这对于任何类型的训练都非常有用。每个图像像素值在转换之前都是这样的：

```python
    # 使用像素值可视化一个数字
    def visualize_input(img, ax):
        ax.imshow(img, cmap='gray')
        width, height = img.shape
        thresh = img.max()/2.5
        for x in range(width):
            for y in range(height):
                ax.annotate(str(round(img[x][y], 2)),
                            xy=(y, x),
                            horizontalalignment='center',
                            verticalalignment='center',
                            color='white' if img[x][y] < thresh else 'black')
    
    fig = plt.figure(figsize=(12, 12))
    ax = fig.add_subplot(111)
    visualize_input(X_train[0], ax)
    plt.show()
```

![](/assets/keras-mnist-for-ios/pixes-daf69647-717f-499d-aa39-fa83904d7675.png)

可以看到图像中的每个灰度像素都是介于0到255之间的，并且当像素为255时，背景色为白色，像素为0时，背景色为黑色。在这里使用的是`mnist.load_data()`加载的数据集，此时并没有对图像进行像素缩放，而在我们自定义的数据集加载方法`mnist_data()`方法中，我们进行了像素的缩放，`X_train = X_train.astype('float32')/255` 。

## 5. One-Hot 编码

最初，数据以Y-Vector包含X Vector（像素数据）包含的数值的方式编码。例如，如果图像看起来像7，那么Y-Vector中必定包含数字7。但是这种方式不利于我们在网络结构中直接使用，我们需要进行这种转换，希望将数据的输出映射到网络中的10个输出神经元，此时当相应的数字被识别时，相应的神经元就会触发，从而达到有效的识别。

![](https://liip.rokka.io/www_inarticle/46a2ef/onehot.png)

## 

## 6. 网络模型化

了解了数据集的基本情况以及进行合理的数据转换后，该是定义卷积神经网络的时候了。这里讲直接使用卷积神经网络中的卷积层和池化层来定义网络，具体实现如下：

```python
    # 定义网络模型
    from keras.models import Sequential
    from keras.layers import Dense, Dropout, Flatten
    from keras.layers import Conv2D, MaxPooling2D
    from keras.optimizers import Adadelta
    
    def network():
        model = Sequential()
        input_shape = (28, 28, 1)
        num_classes = 10
    
        model.add(Conv2D(filters=32, kernel_size=(3, 3), padding='same', activation='relu', input_shape=input_shape))
        model.add(MaxPooling2D(pool_size=2))
    
        model.add(Conv2D(filters=32, kernel_size=2, padding='same', activation='relu'))
        model.add(MaxPooling2D(pool_size=(2, 2)))
    
        model.add(Conv2D(filters=32, kernel_size=2, padding='same', activation='relu'))
        model.add(MaxPooling2D(pool_size=(2, 2)))
    
        model.add(Dropout(0.3))
        model.add(Flatten())
        model.add(Dense(500, activation='relu'))
    
        model.add(Dropout(0.4))
    
        model.add(Dense(num_classes, activation='softmax'))
    
        # 模型概述
        print(model.summary())
        return model
```

在模型的定义中，我们以内核大小为3的[卷积](https://keras.io/layers/convolutional/)，这也意味着窗口为3x3像素，输入形状的大小为28x28像素。紧跟着使用了一个池化大小为2的[池化层](https://keras.io/layers/pooling/)，这里的池化大小为2，意味着将会对每一个输入缩减为原来的一般，因此在下一个卷积层中，输入大小为14x14像素。按照此方式重复两次后，最终的卷积输入大小转换为3x3像素。接下来，使用了[Dropout层](https://keras.io/layers/core/#dropout)，将30%的输入单元随机设置为0，以防止训练的过拟合。最后，展平输入层（此例子中为3x3x32=288），并将它们连接到一个具有500个输入的密度层。在这些步骤之后，添加了另一个Dropout层，之后连接到最后的密度层，该密度层中包含10个输出单元，这些输出单元对应着我们的目标类别，0到9之间的数字。

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    conv2d_1 (Conv2D)            (None, 28, 28, 32)        320       
    _________________________________________________________________
    max_pooling2d_1 (MaxPooling2 (None, 14, 14, 32)        0         
    _________________________________________________________________
    conv2d_2 (Conv2D)            (None, 14, 14, 32)        4128      
    _________________________________________________________________
    max_pooling2d_2 (MaxPooling2 (None, 7, 7, 32)          0         
    _________________________________________________________________
    conv2d_3 (Conv2D)            (None, 7, 7, 32)          4128      
    _________________________________________________________________
    max_pooling2d_3 (MaxPooling2 (None, 3, 3, 32)          0         
    _________________________________________________________________
    dropout_1 (Dropout)          (None, 3, 3, 32)          0         
    _________________________________________________________________
    flatten_1 (Flatten)          (None, 288)               0         
    _________________________________________________________________
    dense_1 (Dense)              (None, 500)               144500    
    _________________________________________________________________
    dropout_2 (Dropout)          (None, 500)               0         
    _________________________________________________________________
    dense_2 (Dense)              (None, 10)                5010      
    =================================================================
    Total params: 158,086
    Trainable params: 158,086
    Non-trainable params: 0
    _________________________________________________________________

## 

## 7. 训练模型

```python
    # 训练模型
    model = network()
    # 编译模型
    model.compile(loss='categorical_crossentropy', optimizer=Adadelta(), metrics=['accuracy'])
    # 使用训练数据拟合模型
    model.fit(X_train, Y_train, batch_size=512, epochs=6, verbose=1, validation_data=(X_test, Y_test))
    # 模型评估分数
    score = model.evaluate(X_test, Y_test, verbose=0)
    
    print('Test loss:', score[0])
    print('Test accuracy:', score[1])
```

这里使用了`categorical_crossentropy`作为损失函数，因为我们的目标类别有多个（0至9），Keras库提供了多种[优化器](https://keras.io/optimizers/#usage-of-optimizers)，你可以选择任意一个进行模型训练，并最终找到一个最好的。经过尝试之后，这里选择[`AdaDelta`](https://keras.io/optimizers/#adadelta)作为优化器进行模型训练，当然你也可以尝试AdaDelta的高级版[AdaGrad](https://keras.io/optimizers/#adagrad)。

![](https://liip.rokka.io/www_inarticle/42b4b8/train.png)

可以看到，经过训练，所得到的模型识别准确率达到了98%，考虑到这里仅仅使用了简单的网络结构，达到这样的准确率已经是非常出色了。在上述截图中，每次迭代的准确性都是在提高，可以说明这里使用的简单结构是合理的，训练得到的模型可以很好地预测输入28x28像素所表示的数字。

## 8. 保存模型

由于我们想要在iOS设备上使用该模型，因此需要将该模型转换为iOS系统能够理解的格式。实际上，微软、Facebook以及亚马逊等企业已经研发出了一套能够在所有深度学习网络格式见转换的协议，以便能够在任何设备上使用的可交换的开放式神经网络交换格式——[ONNX](https://onnx.ai/)。

但是，截止目前，Apple设备上仅仅能够使用的是CoreML格式。为了能够将Keras模型转换为CoreML格式，Apple特意推出来一个非常方便的帮助类库——[coremltools](https://apple.github.io/coremltools/generated/coremltools.converters.keras.convert.html)，这里我们就可以使用该类库来完成工作。该类库能够将scikit-learn、Keras、XGBoost等机器学习类库训练的模型转换为CoreML支持的格式，从而使得模型能够直接在Apple设备上使用。如果你还未安装coremltools类库，可以使用`pip install coremltools`进行安装，然后再使用。

```python
coreml_model = coremltools.converters.keras.convert(model,
                                                        input_names="image",
                                                        image_input_names='image',
                                                        class_labels=['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
                                                        )
```

在进行模型转换的时候，最重要的参数是class_labels，它定义了模型尝试预测的类数，以及input_names或者image_input_names。通过将它们设置为图像，Xcode会自动识别该模型是关于接收图像并从中预测某些内容，也就是说这些参数是告诉Xcode，该模型是关于那方面的任务。根据应用程序和模型的特定功能，需要研究[官方文档](https://apple.github.io/coremltools/generated/coremltools.converters.keras.convert.html)进一步的了解这些参数的可选值等。

另外还有一些可以定义模型元信息的参数，这些参数可以给模型一个简要的说明，甚至作者、license等，可以让使用者能够方便的查阅模型所针对的特定任务等。

```python
    # 编辑模型元信息
    coreml_model.author = 'Robin'
    coreml_model.license = 'MIT'
    coreml_model.short_description = 'MNIST handwriting recognition with a 3 layer network'
    coreml_model.input_description['image'] = '28x28 grayscaled pixel values between 0-1'
    coreml_model.save('SimpleMnist.mlmodel')
    
    print(coreml_model)
```

## 9. 使用模型预测

在将模型保存为CoreML格式之后，我们可以尝试使用转换后的模型进行一个预测，来确定模型是否工作正常。在这里我们将从MNIST数据集中选择一张图像进行预测验证。

 ```python
   # 使用CoreML模型预测验证
    from PIL import Image  
    import numpy as np
    model =  coremltools.models.MLModel('SimpleMnist.mlmodel')
    im = Image.fromarray((np.reshape(mnist_data()[0][0][12]*255, (28, 28))).astype(np.uint8),"L")
    plt.imshow(im)
    predictions = model.predict({'image': im})
    print(predictions)
 ```

输出结果：

    {u'classLabel': u'3', 
    u'output1': {u'1': 0.0, 
    						u'0': 0.0, 
    						u'3': 1.0, 
    						u'2': 0.0, 
    						u'5': 0.0, 
    						u'4': 0.0, 
    						u'7': 0.0, 
    						u'6': 0.0, 
    						u'9': 0.0, 
    						u'8': 0.0
    						}
    }

![](/assets/keras-mnist-for-ios/download-45f07bef-9ca6-4ea6-a674-789607207e9c.png)

可以看到，预测过程和结果均符合预期。接下来是时候在Xcode项目中使用该模型了。

# 10步完成模型在Xcode项目中的应用

为了能够让几乎所有人了解机器学习模型文件是如何一步一步在Xcode项目中使用的，这里将会从最为基础的Xcode安装、项目创建等说起，如果你是iOS开发的老鸟，部分内容请自行略过。

## 1. 安装Xcode

对于iOS体系来说，Xcode是开发iOS应用程序必须的工具之一，因此如果你还未安装Xcode，需要安装Xcode，最为简单的方式是在Mac App Store中搜索并安装。如果你已经安装了Xcode，需要确保Xcode的版本至少在9.0或以上。

## 

## 2. 创建项目

安装好Xcode之后，开启Xcode，选择iOS平台下的单视图应用，命名项目，这里命名为“MNIST-Demo”，选择一个保存项目文件的位置，创建项目即可。

![](/assets/keras-mnist-for-ios/Untitled-78fa75ca-4376-49e8-b194-bde1699f9be3.png)

## 3. 添加CoreML模型文件

现在，你可以将通过coremltools转换得到的CoreML模型加入到项目中了。最简单的方式是直接拖拽模型文件到项目目录中，如果为了之后更新模型而不用去删掉重新添加，你可以在弹出的选项框中选择“add as Reference”。

![](/assets/keras-mnist-for-ios/add-model-98101cf6-8cf8-4d2a-b58e-30d397c98354.png)

## 4. 删除不需要的视图或者故事版

因为我们仅仅使用相机并显示标签，因此这里会删除掉项目中默认的一些用户界面，也就是项目中的视图控制器和故事面板。当然你也可以选择不删除，直接使用现有的视图和故事面板进行开发，不论选择哪种方式都能达到目的。这里要注意的是，如果选择编码的方式构建应用，再删除了主故事面板文件后，需要在项目的TARGETS中同步删除"Main Interface"的默认设置。

![](/assets/keras-mnist-for-ios/ScreenShot2018-11-22at10-9150b677-f351-4732-bfb6-d74019527380.32.46AM.png)

## 5. 程序化创建根视图控制器

接下来我们将使用代码的方式，重新制定应用程序的根视图。具体如下：

```swift
    // 通过编码的方式指定根视图控制器
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
         // 创建窗口
         window = UIWindow()
         window?.makeKeyAndVisible()
            
         // 指定根视图控制器
         let vc = ViewController()
         window?.rootViewController = vc
            
         return true
    }
```

## 6. 构建视图控制器细节

接下来就是构建视图控制器的详细内容细节了。我们需要以下可交互的元素组件，例如按钮，也需要作为展示结果或者状态的标签等，另外重要的是，由于需要使用相机，因此AVFoundation类库是必须要添加的，该库用来访问和控制iOS设备上的相机，还需要Vision库，该库是iOS推出的用于计算机视觉相关任务的工具库，能够很好的和CoreML模型之间进行交互等。

具体的代码细节，这里不再累述，完成之后的代码如下：

```swift
    // 定义视图控制器
    import UIKit
    import AVFoundation
    import Vision
    
    // 由于要使用到相机设备进行视频流的输入，因此这里要继承AVCaptureVideoDataOutputSampleBufferDelegate协议
    class ViewController: UIViewController, AVCaptureVideoDataOutputSampleBufferDelegate {
        // 创建一个文本标签用来显示识别结果
        let label: UILabel = {
            let label = UILabel()
            label.textColor = .white
            label.translatesAutoresizingMaskIntoConstraints = false
            label.text = "Label"
            label.font = label.font.withSize(40)
            return label
        }()
    
        override func viewDidLoad() {
            // 调用相机设备设置方法、文本标签设置方法
            super.viewDidLoad()       
            setupCaptureSession()
            view.addSubview(label)
            setupLabel()
        }
    
            // 设置相机设备session
        func setupCaptureSession() {
            // 创建一个新的捕获session
            let captureSession = AVCaptureSession()
    
            // 查找可用的相机设备
            let availableDevices = AVCaptureDevice.DiscoverySession(deviceTypes: [.builtInWideAngleCamera], mediaType: AVMediaType.video, position: .back).devices
    
            do {
                // 选择首个设备并设置为输入源
                if let captureDevice = availableDevices.first {
                    captureSession.addInput(try AVCaptureDeviceInput(device: captureDevice))
                }
            } catch {
                // 如果未找到相机设备，则打印错误信息
                print(error.localizedDescription)
            }
    
            // 将视频输出设置到屏幕并将输出添加到我们的捕获会话
            let captureOutput = AVCaptureVideoDataOutput()
            captureSession.addOutput(captureOutput)
            let previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
            previewLayer.frame = view.frame
            view.layer.addSublayer(previewLayer)
    
            // 缓冲视频并启动捕获会话
            captureOutput.setSampleBufferDelegate(self, queue: DispatchQueue(label: "videoQueue"))
            captureSession.startRunning()
        }
    
        func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
            // 加载Core ML 模型
            guard let model = try? VNCoreMLModel(for: SimpleMnist().model) else { return }
    
            // 使用Core ML运行推理
            let request = VNCoreMLRequest(model: model) { (finishedRequest, error) in
    
                // 捕获推理结果
                guard let results = finishedRequest.results as? [VNClassificationObservation] else { return }
    
                // 捕获得分最高的推理结果
                guard let Observation = results.first else { return }
    
                // 构建最终显示的文本格式
                let predclass = "\(Observation.identifier)"
    
                // 显示在文本标签内
                DispatchQueue.main.async(execute: {
                    self.label.text = "\(predclass) "
                })
            }
    
              // 创建一个核心视频像素缓冲区，它是一个图像缓冲区，用于保存主存储器中的像素生成帧，
                    // 压缩或解压缩视频或使用核心图像的应用程序都可以使用核心视频像素缓冲区
            guard let pixelBuffer: CVPixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
    
            // 执行请求
            try? VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:]).perform([request])
        }
    
        func setupLabel() {
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
            label.bottomAnchor.constraint(equalTo: view.bottomAnchor, constant: -50).isActive = true
        }
    }
```

> 如果你直接使用上述代码，请记得修改模型的名称。

![](/assets/keras-mnist-for-ios/Untitled-503351b4-b11a-4e7b-a7ad-de4017cbac28.png)

## 6. 添加隐私说明信息

由于我们要使用相机设备进行视频数据流的获取，因此需要在Xcode工程项目中的info.plist文件中添加相应的权限申请说明“*Privacy - Camera Usage Description*”，并附带相应的说明性文字：

![](/assets/keras-mnist-for-ios/Untitled-8e6fa826-e368-47d0-8256-588a136119db.png)

## 7. 加入苹果开发者计划

为了能够让该应用程序运行在你的手机设备上，你可能需要注册[苹果的开发者计划](https://developer.apple.com/programs/enroll/)。当然如果你不想为了运行项目而花费金钱，你也可以按照[此教程](https://9to5mac.com/2016/03/27/how-to-create-free-apple-developer-account-sideload-apps/)注册免费的账户。

## 8. 在iPhone设备上发布应用

一切准备好之后，你就可以将该应用程序发布到你的手机设备上了。你可以按照如下图所示的方式发布项目，也可以直接在Xcode中选定目标设备，然后使用快捷键CMD+R的方式构建：

![](/assets/keras-mnist-for-ios/Untitled-1a74e4bf-8c84-4e49-95f2-4b46f8f4102f.png)

## 9. 使用应用程序

经过上述各种设置和编码之后，终于可以在设备上运行我们的应用程序了。如果一切正常，首次应用程序启动的时候，会询问你是否允许应用程序访问设备的相机，这里需要允许，否则我们的应用程序则无法正常工作。

另外，我们这里所训练的模型以及制作的应用程序，没有进行详细的设计和优化，在识别的过程中，可能会遇到识别不出来以及识别错误的情况，如果需要将此功能应用在你的产品中，需要严格重新审查你所拥有的数据，以及模型的训练，app的使用等，以免出现不可预知的错误等问题。

![](https://liip.rokka.io/www_inarticle/812493/output.gif)

# 总结

通过这篇文章，希望能够让你了解如何使用Keras训练所需要的模型，以及如何将其应用在iOS平台下的应用程序中，虽然介绍的不够深入，但是希望能够带给你继续深入理解Keras、了解Core ML的欲望，早日在你的应用程序中实现AI的能力，为你的应用程序增添色彩。
