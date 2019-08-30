---
layout: post
author: Robin
title: 在Swift中获取类的属性列表 --- 没有runtime也能做到
tags: 开发知识 iOS
categories:
  - 开发知识
---


> 此文中的方法仅针对继承 `NSObject`的子类！
> 仅记录一些代码段而已。 囧



在数据解析或者数据检查的世界里，常常会使用到一些所谓的“黑魔法”技术，但是这些技术又能够很好的解决所遇到的问题，也算是功不可没了。  
  
在iOS的世界里，此类技术的核心使用的是`Runtime`特性。关于`Runtime`的介绍，可以参考苹果公司的[Runtime开发指南文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)。  
  
### 需求  
  
存在一个继承自**NSObject**的类，其中有一些属性。需要获取到此类的所有属性列表，并将其属性的值保存在一个[key : value] 中。
 
### 代码实现

```
extension NSObject {

	// Retrieves an array of property names found on the current object
    func propertyNames() -> [String] {
        var results = [String]()
        
        // retrieve the properties via the class_copyPropertyList function
        var count: UInt32 = 0
        let myClass: AnyClass = self.classForCoder
        let properties = class_copyPropertyList(myClass, &count)
        
        // iterate each objc_property_t struct
        for i in 0 ..< count {
            let property = properties[Int(i)]
            let cname = property_getName(property)
            
            //convert the c string into a Swift string
            let name = String.fromCString(cname)
            results.append(name!)
        }
        
        
        return results
    }
}
```

### 使用

假设待测试的类如下：

```
class Object: NSObject {
    var number = 0
    var string = "string value"
    var array  = ["sss", "lslslsl"]
}
```

那么获取此类的属性列表就是：

```
let obj = Object()
let properties = obj.propertyNames()
```

进一步，获取属性的值就是：

```
obj.valueForKey("string") // string value
```