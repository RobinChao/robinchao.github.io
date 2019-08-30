---
layout: post
author: Robin
title: 如何计算iOS设备的倾斜量 --- iOS加速度传感器数据yaw数据的获取
tags: 开发知识 传感器 iOS
categories:
  - 传感器
  - 开发知识
--- 

设备的倾斜量对于获取iOS设备，比如iPhone、iPad、iPod Touch等设备的加速度数据非常有用。比如在一些游戏中，经常会遇到需要倾斜设备去进行游戏的方式。但是问题是，**如何获取设备的倾斜量呢？**

#### 初步想法 

苹果在其官方文档中给出了[CoreMotion](http://developer.apple.com/library/ios/#documentation/CoreMotion/Reference/CoreMotion_Reference/_index.html)的完整文档，苹果也为我们已经计算好了一些数据，设备姿态的数据在`CMAttitude`类中。

> An instance of the CMAttitude class represents a measurement of the device’s attitude at a point in time.

> [CMAttitude reference](http://developer.apple.com/library/ios/#documentation/CoreMotion/Reference/CMAttitude_Class/Reference/Reference.html)

但是苹果并没有告知我们设备的详尽数据，类似[芭蕾舞演员姿态](http://c.hiphotos.baidu.com/zhidao/pic/item/b58f8c5494eef01f3045a699e0fe9925bc317d1c.jpg)那样的数据，更多的是[飞行姿态型](http://en.wikipedia.org/wiki/Flight_dynamics)数据。在iOS设备上，此类数据描述为飞行器的`roll`、`pitch`、`yaw`。

![](/assets/iphone-attitude.png)

你有可能已经猜到了，`yaw`的数据就是上图中红色线表示的方向的旋转，感觉可以直接获取这个数据就可以了，OK，我们来实现一下：

{% highlight objc  %}
- (void)viewDidLoad {
    [super viewDidLoad];

    self.motionManager = [[CMMotionManager alloc] init];
    self.motionManager.deviceMotionUpdateInterval = 0.02;  // 50 Hz

    self.motionDisplayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(motionRefresh:)];
    [self.motionDisplayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];

    if ([self.motionManager isDeviceMotionAvailable]) {
        // to avoid using more CPU than necessary we use `CMAttitudeReferenceFrameXArbitraryZVertical`
        [self.motionManager startDeviceMotionUpdatesUsingReferenceFrame:CMAttitudeReferenceFrameXArbitraryZVertical];
    }
}

- (void)motionRefresh:(id)sender {
    double yaw = self.motionManager.deviceMotion.attitude.yaw;

    // use the yaw value
    // ...
}
{% endhighlight %} 

直接就获取到了`yaw`的数据，貌似“简单，问题解决。”，但是事实上，这个数据是不可直接使用的，因为`roll`和`pitch`会直接影响`yaw`的数据。意思就是说，当我们保持设备垂直于上图中的蓝色线，那么`yaw`值会改变。


如果需要了解更多相关的知识，可以查看[万向节死锁](http://en.wikipedia.org/wiki/Gimbal_lock)。


#### 四元数的美

**如果你不知道四元数是什么？那就去好好的看一下《Star Trek》吧。**

> Quaternions were first described by Hamilton in 1843 and applied to mechanics in three-dimensional space.

> [Wikipedia](http://en.wikipedia.org/wiki/Quaternion)

它简化了在三维空间中定位身体的方向的方式，而且更加的适合[比欧拉角](http://en.wikipedia.org/wiki/Euler_angles)。苹果提供了强大的计算函数以供开发者直接使用，为开发者带来的福利可能有一下三点：

* 能够更加容易的编写旋转函数，并从中获取数据。
* 避免了万向节锁定问题。
* 提供了一个`CMAttitude`实例的四元数以供直接使用。


因为我们只需要计算`yaw`，因此不用担心有万向节死锁的问题。而且我们的目标也不是去定位设备在三维空间中的方向，仅仅需要关心的是设备的倾斜就可以了。

从四元数中计算`yaw`分量有一个简单的方式：

![](/assets/tilt.png)

因此，`motionRefresh：`方法经过改进就如下：

{% highlight objc  %}
- (void)motionRefresh:(id)sender {
    CMQuaternion quat = self.motionManager.deviceMotion.attitude.quaternion;
    double yaw = asin(2*(quat.x*quat.z - quat.w*quat.y));

    // use the yaw value
    // ...
}
{% endhighlight %} 

#### 锦上添花

{% highlight objc  %}
- (void)motionRefresh:(id)sender {
    CMQuaternion quat = self.motionManager.deviceMotion.attitude.quaternion;
    double yaw = asin(2*(quat.x*quat.z - quat.w*quat.y));

    if (self.motionLastYaw == 0) {
        self.motionLastYaw = yaw;
    }

    // kalman filtering
    static float q = 0.1;   // process noise
    static float r = 0.1;   // sensor noise
    static float p = 0.1;   // estimated error
    static float k = 0.5;   // kalman filter gain

    float x = self.motionLastYaw;
    p = p + q;
    k = p / (p + r);
    x = x + k*(yaw - x);
    p = (1 - k)*p;
    self.motionLastYaw = x;

    // use the x value as the "updated and smooth" yaw
    // ...
}
{% endhighlight %} 

以上的方式使用了[卡尔曼滤波](https://en.wikipedia.org/wiki/Kalman_filter)将数据进行了过滤处理，这样使得最终的数据能够呈现线性的方式，而无太多的抖动。