---
layout: post
author: Robin
title: 如何获取iOS设备的法线向量 --- 四元数的辉煌绽放
tags: 传感器 开发知识 iOS
categories:
  - 传感器
  - 开发知识
--- 

在iOS的传感器系统中，`CMAttitude`变量可以得到多个方向上的分量数据，其中`z`向分量是垂直于设备屏幕的，类似下图:

![](/assets/attitude-z.png)

在上图中，可以看到不管设备如何旋转，哪怕是设备被高高悬挂在高空，只要屏幕指向地心方向，那么`z`向的数据应该就是`(0,-1,0)`，而且保持不变，即使是设备绕着地心像直升机螺旋桨那样旋转。

![](/assets/attitude-rotation.png)

对于这种分量数据的保持，其实就使用到了上篇[如何获取iOS世界坐标系中的加速度 --- 关于坐标系的转换](https://robinchao.github.io/blog/2016/09/motion-relative-to-world)中提到的设置制定的参考系了，由于仅仅需要获取`z`向的数据，因此就直接获取参考系为`CMAttitudeReferenceFrameXTrueNorthZVertical`。获取到分量数据后，就需要使用四元数进行矩阵旋转了（[旋转矩阵](http://content.gpwiki.org/index.php/OpenGL:Tutorials:Using_Quaternions_to_represent_rotation#Rotating_vectors)）。


使用的公式就是`n = q * e * q'`，`q`表示CMAttitude[w,(x,y,z)]，`q'`表示共轭矩阵[w,(-x,-y,-z)]，`e` 是面朝上正常 [0，(0，0，1)] 的四元数表示形式。虽然可以使用苹果提供的`CMQuaternion`[Link](https://developer.apple.com/library/ios/documentation/CoreMotion/Reference/CMAttitude_Class/#//apple_ref/c/tdef/CMQuaternion)进行计算，但是还是比较的繁琐，google有一个辅助工具类[cocoamath](http://code.google.com/p/cocoamath/)，可以很方便的进行这样的计算。


{% highlight objc  %} 
Quaternion e = [[Quaternion alloc] initWithValues:0 y:0 z:1 w:0];
CMQuaternion cm = deviceMotion.attitude.quaternion;
Quaternion quat = [[Quaternion alloc] initWithValues:cm.x y:cm.y z:cm.z w: cm.w];
Quaternion quatConjugate = [[Quaternion alloc] initWithValues:-cm.x y:-cm.y z:-cm.z w: cm.w];
[quat multiplyWithRight:e];
[quat multiplyWithRight:quatConjugate];
// quat.x, .y, .z contain your normal
{% endhighlight %}

Quaternion.h:

{% highlight objc  %} 
@interface Quaternion : NSObject {
    double w;
    double x;
    double y;
    double z;
}

@property(readwrite, assign)double w;
@property(readwrite, assign)double x;
@property(readwrite, assign)double y;
@property(readwrite, assign)double z;
{% endhighlight %}

Quaternion.m:

{% highlight objc  %} 
- (Quaternion*) multiplyWithRight:(Quaternion*)q {
    double newW = w*q.w - x*q.x - y*q.y - z*q.z;
    double newX = w*q.x + x*q.w + y*q.z - z*q.y;
    double newY = w*q.y + y*q.w + z*q.x - x*q.z;
    double newZ = w*q.z + z*q.w + x*q.y - y*q.x;
    w = newW;
    x = newX;
    y = newY;
    z = newZ;
    // one multiplication won't denormalise but when multipling again and again 
    // we should assure that the result is normalised
    return self;
}

- (id) initWithValues:(double)w2 x:(double)x2 y:(double)y2 z:(double)z2 {
        if ((self = [super init])) {
            x = x2; y = y2; z = z2; w = w2;
        }
        return self;
}
{% endhighlight %}

以上只是一些示例代码，通过一些计算之后，就可以得到垂直于设备屏幕的法线向量了。
