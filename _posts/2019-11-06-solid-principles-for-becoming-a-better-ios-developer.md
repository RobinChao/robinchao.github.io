---
layout: post
author: Robin
title: 如何成为更好的iOS开发工程师之S.O.L.I.D原则
tags: 开发知识 技术人生
categories:
- 技术人生
cover: '/assets/solid/cover.jpg'
---

在互联网时代，**S.O.L.I.D**原则可谓影响力久远，在计算机程序设计语言以及各个平台特性中都有S.O.L.I.D的身影，S.O.L.I.D原则也指导着软件工程的设计与编码工程。iOS平台的软件开发亦是软件开发领域的一支，S.O.L.I.D原则也同样对iOS软件开发有效，并且做称为一个更好的iOS软件开发人员，对S.O.L.I.D原则或许要理解更加深刻，并付诸实践。

**S.O.L.I.D**原则本质上是五个面向对象编程（OOP）的指导性原则。当在进行类或者模块的设计和编码时，遵循S.O.L.I.D原则可以让软件更加的健壮和稳定。

* **S**ingle Responsibility Principle（单一职责原则，SRP）
* **O**pen/closed Principle（开放封闭原则，OCP）
* **L**iskov Substitution Principle（里氏替换原则，LSP）
* **I**nterface Segregation Principle（接口隔离原则，ISP）
* **D**ependency Inversion Principle（依赖倒置原则，DIP）

## 单一职责原则（SRP）

**单一职责原则（SRP）**指的是一个类或者模块有且只有一个职责。一个类就像一个容器，它能添加任意数量的属性、方法等。但是，如果视图让一个类实现太多功能，很快这个类就开始变得臃肿笨重。任意小的一个改动都可能导致这个类发生变化，重复的全量测试等等。但是如果遵循SRP，类将保持简洁且灵活的状态，每个类将只负责单一的一个问题、任务或者关注点，这样的方式对于开发测试来说，代价最小。**SRP的核心就是把整个问题拆分成小模块，并且每个小模块都将通过一个单独的类进行实现、负责。**

通常在开发阶段，因为SRP原则的简单，我们很容易违背SRP原则。**最大的现象是小功能或者小特性。**那些小特性往往让开发慢慢陷入困境，特别是在团队作战中，你为一个类添加了一个小特性，另一个人添加了另一个小特性，慢慢的该类的功能开始变得繁多，而到最后，如何使用该类以及优化和重构该类成了最大的纠结。

相对来说，iOS开发人员可能是最容易违背SRP原则的，因为iOS体系的特殊性，**UIViewController**是我们无法避开的。

简单点说，**UIViewController**是将屏幕上的各个视图组合在一起，例如表格视图、图片视图等等，另外**UIViewController**还承担着**UIViewController**之间的导航作用，有时，还可能承担网络请求等等。不完全统计**UIViewController**共有12中职责，这可能是严重违背SRP原则的一个iOS组件，因此在进行iOS软件开发的时候，人们都称App中的**UIViewControllers**为**Massive View Controller**，即大规模视图控制器。

这也是为什么几乎每个iOS开发者都不愿意随意的改变ViewController的地方，由于其承担的责任较多、小特性很多，一个不完整或者考虑不周全的改动，可能导致应用程序无法正常运行或者运行不符合预期等。

### 如何应对呢？

首先，坚决不为了单一的快而在原有的类上添加小功能、小特性，转而思考模块、组件或者API的方式。在开发过程中，需要我们摆脱掉修修补补的思想或者黑客的思维，为了软件的生命完整性和可扩展性，考虑类库形式的解决方案等。构建尽量小的类，只完成一个任务或者只解决一个问题。如果面对的问题是个相对大的问题，试着分解问题成多个小问题，然后为每个小问题编写对应的解决方案类，最终构建一个类来组装各个小类，解决大的问题。

重新审视项目中的ViewController，如果该类过于沉重，试着分解该类中不同功能，让ViewController变的轻量。一个很好的例子是iOS SDK提供的UITableView的组装方式，使用delegate和dataSource分离动作和数据源，让TableView的实现条理分明，简洁快速。**使用Data Source的方式组织数据，是任何类都可以施行的方式，不仅仅只针对ViewController。**

## 开放封闭原则（OCP）

**开放封闭原则（OCP）**指出，一个类应该对扩展开放，对修改关闭。这意味一旦你创建了一个类并且应用程序的其他部分开始使用它，你不应该修改它。为什么呢？因为如果你改变它，很可能你的改变会引发系统的崩溃。如果你需要一些额外功能，你应该扩展这个类而不是修改它。使用这种方式，现有系统不会看到任何新变化的影响。同时，你只需要测试新创建的类。

假设我们有一个获取用户数据的类`UserFetcher`，在该类中有一个方法`fetchUsers`，如下：

```swift
class UserFetcher {
    func fetchUsers(onComplete: @escaping ([User]) -> Void) {
        let session = URLSession.shared
        let url = URL(string: "")!
        session.dataTask(with: url) { (data, _, error) in
            guard let data = data else {
                print(error!)
                onComplete([])
                return
            }
            
            let decoder = JSONDecoder()
            let decoded = try? decoder.decode([User].self, from: data)
            onComplete(decoded ?? [])
        }
    }
}
```

该方法乍一看很好的实现了从网络加载数据，进行解析并返回解析后的数据。但是，假设有另一个任务需要从网络加载`Article`的数据，如果依照上述写法，需要重新构建一个类，用来加载`Article`数据，看似无误，但是问题在于，加载`User`的方法和加载`Article`的方法99%都是相同的，如果要如此重复的写下去，那么代码量将翻倍重复。另一个严重的问题在于，如果加载协议发生了改变，每一个加载数据的类都需要修改，很有可能演变成异常灾难。那么比较好的写法是什么呢？

```swift
class Fetcher<T: Decodable> {
    func fetch(onComplete: @escaping ([T]) -> Void) {
        let session = URLSession.shared
        let url = URL(string: "")!
        session.dataTask(with: url) { (data, _, error) in
            guard let data = data else {
                print(error!)
                onComplete([])
                return
            }
            
            let decoder = JSONDecoder()
            let decoded = try? decoder.decode([T].self, from: data)
            onComplete(decoded ?? [])
        }
    }
}
```

针对数据加载类进行了重构，定义了一个支持任何`Decodable`协议的类`Fetcher`，也就是定义了一个支持泛型的类，改造后的类能够支持所有相同的返回值的数据接在与解析等。例如：

```swift
typealias UserFetcher = Fetcher<User>
typealias ArticleFetcher = Fetcher<Article>
```

**开放封闭原则（OCP）**的良好遵循，能够很好的拯救开发人员的时间，也能够让整个项目快速演进。上述例子可能不足以完整的说明开放封闭原则的重要性，但是在不断地思考和实践的过程中，还是建议有意的将开放封闭原则带入到软件开发的过程中，会有意想不到的好效果。

## 里氏替换原则（LSP）

**里氏替换原则（LSP）**指的是，派生的子类应该是可替换基类的，也就是说任何基类出现的地方，子类一定可以出现。值得注意的是，当你通过继承实现多态行为时，如果派生类没有遵循LSP，可能会使系统出现异常。所有要谨慎使用继承，只有确定是**is-a**关系时才使用继承。另外，LSP表示任何与类一起使用的方法函数也应该与这些类的任何子类一起使用，如果重写方法，该方法的使用者应该看不到基类对应的方法与子类所重写的方法之间的区别。

例如上述例子中，`ArticleFetcher`是从网络加载数据，进行解析和返回结果的，但是某个时刻，`Article`数据可能并不需要从网络进行加载，而是从本地文件系统进行加载，此时良好的解决方案就是重写`fetch`方法，例如：

```swift
class FileFetcher<T: Decodable>: Fetcher<T> {
    override func fetch(onComplete: @escaping ([T]) -> Void) {
        let json = try? String(contentsOfFile: "article.json")
        guard let data = json?.data(using: .utf8) else {
            return
        }
        
        let decoder = JSONDecoder()
        let decoded = try? decoder.decode([T].self, from: data)
        onComplete(decoded ?? [])
    }
}
```

快速的方法重写后，好像都对，但是这里犯了一个严重的错误。基类的工作方式是，如果发生了错误，会返回一个空的数组，完成程序处理，然而重写后的方法如果发生了错误，则什么都不发生。这样对于使用该方法的UI界面则不会更新，也不会有提示等。

```swift
// 方式1
let fetcher = FileFetcher<Article>()
fetcher.fetch { articles in
    self.articles = articles
    self.tableView.reloadData()
}
// 方式2
if fetcher is FileFetcher {
    tableView.reloadData()
}
```

其实这两种方式都是**不对或不严谨**的。无论是上述哪一种方式，最终的目的都是不改变基类的基础上，让子类完整的实现和基类相同的行为，达到目标一致的结果。方式1看似没有问题，但是子类的行为在实现的时候忽略了发生错误时的程序行为，方式2 可以算作是一种偷懒的方式，虽然`fetcher`对象的确是`FileFetcher`，但是这样的方式完全丢弃了构建子类的目的，也失去了子类化的意义，就像使用代理回调和Block回调一样。

## 接口隔离原则（ISP）

**接口隔离原则（ISP）**表明类不应该被迫依赖他们不使用的方法，也就是说一个接口应该拥有尽可能少的行为，接口的实现应该精简且功能单一。假设上述关于`Article`的数据获取之后，在列表中展示后，我们还需要获取用户点击列表项之后，展示详情。作为一个面向协议的程序员，这里可以使用协议的方式解决该问题。

```swift
protocol ArticleFetcher {
    func getArticles(onComplete: ([Article]) -> Void)
    func getArticle(id: String, _: ([Article]) -> Void)
}
```

此时构建一个获取详情的类，并实现`ArticleFetcher`协议。虽然这样可以解决上述问题，但是带来的问题是，在列表页，并不需要`getArticle`，在详情页不需要`getArticles`。上述协议方法的定义方式，提供了不需要的方法，直接增加了混乱和噪声，这也违背了**单一职责原则（SRP）**中讨论的所有问题。

为了解决此问题，可以分解上述协议为两个，提供职责单一，不耦合的协议定义方式，例如：

```swift
protocol ArticlesFetcher {
    func getArticles(onComplete: ([Article]) -> Void)
}

protocol ArticlesFetcher {
    func getArticle(id: String, _: ([Article]) -> Void)
}
```

分开定义后，之前的实现并不需要再次修改，同一个类可以同时实现这两个协议。在列表控制器里，使用`ArticlesFetcher`的实例，而不会造成额外的混乱，这样，不仅可以在获取详情的勒种添加功能，还不会为类的用户带来使用麻烦。

这也是为什么在Swift语言中会有`Decodable、Encodable、Codable`这样的协议。但是这样的设计可能并不符合所有人的设计，也不是每个人都需要的功能。但是良好的设计，符合SRP的设计对软件的稳定性、健壮性更有利。

## 依赖倒置原则（DIP）

**依赖倒置原则（DIP）**表明高层模块不应该依赖底层模块，相反，他们应该依赖抽象类或接口。在模块设计中，不应该在高层模块中使用具体的底层模块。因为这样的话，高层模块将变得紧耦合底层模块。如果改变了底层模块，那么高层模块也会被修改。根据DIP原则，高层模块应该依赖抽象类或者接口，底层模块也是如此。通过面向接口（抽象类）编程，紧耦合被消除。

那么什么是高层模块，什么是低层模块呢？通常情况下，我们会在一个类（高层模块）的内部实例化它依赖的对象（低层模块），这样势必造成两者的紧耦合，任何依赖对象的改变都将引起类的改变。

依赖倒置原则表明高层模块、低层模块都依赖于抽象。如果我们将上述定义的协议称为`Fetchable`协议，那么在视图控制器中使用的应该是`Fetchable`协议，而不是`Fetcher`类。

原因则是**减少耦合。**当一个类严重依赖另一个类的实现时，会发生强耦合，可能会调用很多方法，对类的内部工作做了假设，或者使用了将其绑定到特定类的变量名等。

强耦合带来的直接后果是，代码库的优化和重构难上加难。例如你正在使用`CoreDataService`协议进行数据库的使用，但事后由于业务的发展等原因，你需要改用`RealmService`，此时最好的情况便是视图控制器没有强依赖`CoreDataService`。

解决此问题的最佳实践是，使用同样的基协议，例如`DatabaseService`，再构建不同的数据库工具类，以实现该协议。

```swift
protocol DatabaseService {
    func getUsers() -> [User]
}

class CoreDataService: DatabaseService {
    // ...
}


let databaseService: DatabaseService = CoreDataService()
```

在视图控制器中使用协议实例，是因为协议比类要少。一个类会有一些特定的名称和特定的方法。另外，**协议是抽象的**。多个类可以实现同一个协议，使其成为减少耦合的理想选择。

如果要切换到`RealmService`，需要做的就是创建一个符合相同协议的类，因为并没有依赖任何特定的实现，所有不需要在试图控制器中修改代码，节省大量时间。

在软件开发的过程中，最好是对代码的组织进行提前思考，将**低耦合，高内聚**在每一次实现中有所体现，最终软件的稳定性和健壮性会为你带来良好的效果。

## 总结

 以上便是**S.O.L.I.D**原则，我们完整从回归了五个重要的软件开发中的最佳实践，但是要说明的是，这些原则虽然非常有用，但是它们不是规则，它们是帮助你提高开发效率、增强软件稳定性、健壮性的工具。**S.O.L.I.D**原则的创造者罗伯特·C·马丁（Robert C. Martin）指出：“他们的陈述是'每天要吃一个苹果，才能远离医生'。”因此，请记住它们，但要妥协。

 **Happy coding!**