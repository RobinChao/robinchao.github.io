---
layout: post
author: Robin
title: 如何优雅的使用Swift Exception
tags: 开发知识 iOS
categories:
  - 开发知识
  - Swift
---

***

 `异常处理机制`是很多现代编程语言必备的特性，而且基本上都使用了`do/try/catch`语法。如果没有使用异常捕捉语句，编译器（预编译器/静态分析器）会告诉你。但是在Swift中，在没有异常处理或抛出，如果不能像Java一样使用异常控制流结构，那当你编写库的代码执行了一个失败的子程序时，可能就会有意想不到的情况被抛出。

在Swift中，我们知道有一个关键词`throws`，可以用来编辑函数方法。`throws`其实是一个变异体，它返回`enum`类型的数据结构，包含两个属性`<T1, T2>`，T1 表示成功与否的结果，T2 表示错误类型，但是这些对于开发者都是隐藏的。

幸运的是，你不用维护一个包含所有异常的列表，而可以向JAVA一样，对函数进行throws标记即可。

> 1. 在Swift中，不能调用一个抛出未知异常的函数。
> 2. 不会引起显著的开销，而且响应迅速。

##### 如何使用Exception来保持代码清洁呢？

假如需要进行一个APP中用户登录部分的数据验证模块。你可能会创建一个新的target，并使用自定义Exception来实现。

首先，定义假设一种错误/异常：假如用户使用了空的用户名和密码来登录。良好的体验应该是，针对错误类型，进行合理的提示用户。

```swift
enum LoginError: ErrorType {
    case EmptyUsername
    case EmptyPassword
}
```

> Swift中自定义错误/异常枚举，需要遵循`ErrorType`协议.


定义好了设想的错误类型后，就可以进行用户登录数据验证函数的定义，并在其中使用LoginError枚举了。

```swift
func loginUserWithUserNmae(username: String?, password: String?) throws -> String {
    
    guard let username = username where username.characters.count != 0 else {
        throw LoginError.EmptyUsername
    }
    
    guard let password = password where password.characters.count != 0 else {
        throw LoginError.EmptyPassword
    }
    
    return "token: " + username
}
```

在函数方法的末尾，使用了关键词`throws`进行了标记。接下来就可以使用`do/try/catch`语法进行异常捕捉了。

```swift
func login() {
    
    do {
        let token = try loginUserWithUsername("konrad1977", password: nil)
        print("user logged in \(token)")
    } catch LoginError.EmptyUsername {
        print("empty username")
    } catch LoginError.EmptyPassword {
        print("empty password")
    } catch {
        print(error)
    }
}
```

现在，就可以捕捉了用户名或者密码为空的异常了，但是这样并没有提示用户什么地方错了。我们希望在发生错误的时候用，直接将异常信息显示出来，并且逻辑操作函数中，不希望看到异常类型和异常信息。这就需要进一步的对LoginError进行扩展实现了。

```swift
extension LoginError: CustomDebugStringConvertible {
    var debugDescription: String {
        switch self {
        case .EmptyUsername:
            return "Username cannot be empty"
        case .EmptyPassword:
            return "Password cannot be empty"
        }
    }
}
```

> 继承`CustomDebugStringConvertible `协议是为了能够重载description函数。

接下来改造登录入口函数，让其更加的简洁。

```swift
func login() {
    do {
        let token = try loginUserWithUserNmae("nameofuser", password: nil)
        print("token: \(token)")
    } catch let error as LoginError {
        print(error.debugDescription)
    } catch {
        print("error")
    }
}
```


这样就很容易的捕捉预想的哪两种错误了。如果还有其他需要捕捉的异常，就可以直接完善LoginError枚举和它的扩展了。对于登录逻辑入口来说，代码是不用变化了。