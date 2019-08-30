---
layout: post
author: Robin
title: 如何获取iOS世界坐标系中的加速度 --- 关于坐标系的转换
tags: 传感器 iOS 开发知识
categories:
  - 传感器
  - 开发知识
--- 


iOS传感器的数据有很多的维度和变量，但是都是基于设备的坐标系得到的，在实际的使用过程中，需要参考世界坐标系来进行设备姿态的识别和定位，就需要将传感器的坐标系，或者成为iOS设备的坐标系转换为真实世界的坐标系，以便在后续的使用中能够和世界坐标系相对应。

iOS的加速度传感器能够提供很多有用的数据，比如`rotationMatrix`和指向三轴方向的分量`x`、`y`、`z`。iOS SDK提供的传感器管理类`CMDeviceMotion`用来获取大多数的传感器数据，对于加速度来说，主要是`userAccelerate`分量数据。


{% highlight objc  %} 
import CoreMotion
var motionManager = CMMotionManager()
...
motionManager.startDeviceMotionUpdatesUsingReferenceFrame(CMAttitudeReferenceFrameXTrueNorthZVertical, toQueue: NSOperationQueue.currentQueue(), withHandler: {
    ...
    var acc: CMAcceleration = deviceMotion.userAcceleration
    var rot = deviceMotion.attitude.rotationMatrix
    self.ax = (acc.x*rot.m11 + acc.y*rot.m21 + acc.z*rot.m31)*9.81
    self.ay = (acc.x*rot.m12 + acc.y*rot.m22 + acc.z*rot.m32)*9.81
    self.az = (acc.x*rot.m13 + acc.y*rot.m23 + acc.z*rot.m33)*9.81
})
{% endhighlight %}

使用`startDeviceMotionUpdatesUsingReferenceFrame`方法可以设置参考系，这个参考系只对`CMDeviceMotion`中的`attitude`变量有效。也就是说在Block中得到的`attitude`是相对于设定的参考系的，而参考系的设定可以根据自己的需求来进行，通常有一下集中可设定的值：

{% highlight objc  %} 

/*
 *  CMAttitudeReferenceFrame
 *  
 *  Discussion:
 *    CMAttitudeReferenceFrame indicates the reference frame from which all CMAttitude
 *        samples are referenced.
 *
 *    Definitions of each reference frame is as follows:
 *        - CMAttitudeReferenceFrameXArbitraryZVertical describes a reference frame in
 *          which the Z axis is vertical and the X axis points in an arbitrary direction
 *          in the horizontal plane.
 *        - CMAttitudeReferenceFrameXArbitraryCorrectedZVertical describes the same reference
 *          frame as CMAttitudeReferenceFrameXArbitraryZVertical with the following exception:
 *          when available and calibrated, the magnetometer will be used to correct for accumulated
 *          yaw errors. The downside of using this over CMAttitudeReferenceFrameXArbitraryZVertical
 *          is increased CPU usage.
 *        - CMAttitudeReferenceFrameXMagneticNorthZVertical describes a reference frame
 *          in which the Z axis is vertical and the X axis points toward magnetic north.
 *          Note that using this reference frame may require device movement to 
 *          calibrate the magnetometer.
 *        - CMAttitudeReferenceFrameXTrueNorthZVertical describes a reference frame in
 *          which the Z axis is vertical and the X axis points toward true north.
 *          Note that using this reference frame may require device movement to 
 *          calibrate the magnetometer.
 */
typedef NS_OPTIONS(NSUInteger, CMAttitudeReferenceFrame) __TVOS_PROHIBITED {
	CMAttitudeReferenceFrameXArbitraryZVertical = 1 << 0,
	CMAttitudeReferenceFrameXArbitraryCorrectedZVertical = 1 << 1,
	CMAttitudeReferenceFrameXMagneticNorthZVertical = 1 << 2,
	CMAttitudeReferenceFrameXTrueNorthZVertical = 1 << 3
};
{% endhighlight %}


在上面获取数据的实例中，使用的是x轴指向地球北极，z轴指向地心，也就是地理上的地心方向。


![](/assets/attitude_world.png)
![](/assets/attitude_halfworld.gif)


通过`attitude`得到一个旋转向量`rot`，然后只要将原来相对于设备的参考系向量旋转到世界参考系中就可以了。

这个旋转使用了矩阵`v_world = M^-1 * v_device = M^T * v_device`。这里的`M`就是旋转矩阵`rot`，因为矩阵旋转的特性，一个矩阵的转置矩阵和逆矩阵是相等的。

而在计算的使用分量都乘以了一个`9.81`，其实这个就是物理上的`g`，所有的加速度都是以g为单位的。

___

##### 参考资料

* [Core Motion Framework Reference](https://developer.apple.com/library/ios/documentation/CoreMotion/Reference/CoreMotion_Reference/index.html#//apple_ref/doc/uid/TP40009686)

* [DeviceMotion relative to world](http://stackoverflow.com/questions/7950096/devicemotion-relative-to-world-multiplybyinverseofattitude)
