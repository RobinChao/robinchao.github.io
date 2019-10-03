---
layout: post
author: Robin
title: 浅谈iOS架构模式
tags: 开发知识 iOS
categories:
- 开发知识
cover: '/assets/iOSArchitecturePatterns/cover.jpeg'
---

每一个软件开发者在开始学习软件开发的时候，可能都不清楚软件的架构设计是什么样的，仅仅是依靠前人的方式进行代码开发的，至少本人是这样的。慢慢熟悉了软件开发后，对于软件代码如何更加合理的进行组织，以前的开发为什么是那样进行组织的便有了有些理解。其实这一切都是软件的架构模式。

对于iOS开发者来说，几乎每个人都熟悉应用程序的测试、代码的重构和通过视图控制器对业务进行支持等，但是如何合理的选用对当前产品业务更加合理的软件架构，往往会被忽略。这里针对当前业界常见的五种架构模式，进行详细的分析和试用，了解每种架构模式。

> 架构模式并不是所有问题都适用的解决方案，它们仅仅描述了移动应用程序代码的组织方式和方法，具体的实现细节往往会跟随业务的变化而变化。

在本文中，将介绍以下五种iOS端的通用架构模式：

* 传统MVC
* 苹果的MVC
* MVP
* MVVM
* VIPER

## 传统MVC

在70年代后期，Model、View、Controller的模式在编程语言Smalltalk-80中出现，随着时间的推移，人们对MVC有了许多不同的理解，尽管最初的想法逐渐被人们遗忘，但是MVC带给软件开发行业的巨大变化是有目共睹的，有必要好好了解一下最初的MVC以及相关的原理等。MVC最初要解决的问题是：**将组件的职责明确划分为模型、视图、控制器。**

**模型：** 一组封装特定主题领域数据及其验证算法的类。 在传统MVC中，模型还包含处理逻辑（“业务逻辑”）。 有两种类型的模型：主动模型和被动模型。 主动模型能够通知其状态的更改（通常是通过观察者模式）。 传统MVC实现被认为是主动模型，该模型对View和Controller一无所知，并且可以独立运行。 在测试中，此要求起着重要作用。

**视图：** 负责（但不一定）显示数据的图形类。 在传统MVC中，仅在只读模式下，视图可以直接访问模型，视图不应直接更改模型的状态，状态的更改应该是控制器的职责。

**控制器：** 直接和外部互动的组件，根据不同的外部行为，控制器会执行一些逻辑，包括但不限于改变模型的状态等，但是控制器不会直接对视图做出响应，也不会保持视图的状态，也就是说控制器并不是视图和模型的中间介质，也不负责将数据从模型输出到视图。


![](/assets/iOSArchitecturePatterns/classicMVC.png)


### 传统MVC原理

在70年代，MVC模式基本上都是在具有实体按键的设备上应用的。一些外部按键事件，该事件和控制器进行交互，控制器决定如何处理该事件。例如，控制器可以更改模型的状态（一般是调用模型的方法），但是绝不能更改视图的状态，仅仅只有模型会直接影响视图。

如果模型的状态发生了更改，模型将通知视图进行相应的更改，并且视图应该读取新的模型数据，然后在必要时更新并重新绘制视图（视图观察者模型）。虽然MVC在控制台模式下成功完成了任务，但图形界面和鼠标或触摸变得越来越流行，用户现在可以直接与视图进行交互，并且视图会生成事件，从理论上讲，该事件应由Controller处理。 实际上传统的MVC已经发生了变化。

在图像界面时代，界面上将要显示各种样式的图形组件，开发人员的大部分任务演变成了建立各个小组件的层级结构并将事件从组件上重定向到所需的类，因此在现代开发中，可以认为视图是由小部件的不同层级结构构成的。

图形组件通常相对比较复杂。例如，UIKit库中的常用按钮（UIButton）可以为按钮的每种状态包含不同的文本（例如，“highlighted” –“处于突出显示状态”，以及“selected” –“处于选定状态”）。 您还可以设置每种状态的文本颜色，可以直接在可视编辑器中进行配置，也可以通过写代码的方式配置。

因此，按钮本身具有设置功能，并且本身也响应外部事件。 实际上，它包含自己的模型（所谓的View Model）和自己的Controller。 因此，当前程序更像是View，Controllers和Model的复杂层次结构。

### 传统MVC的缺点

传统MVC的缺点之一是组件之间的强互连性，这使单元测试变得复杂。 在现代程序中，控制器，视图和视图模型的层次结构愈发复杂，它们被认为是基于MVC的应用程序，因此实际上无法进行单元测试。

另一个问题是业务模型的“增厚”。 为什么会这样呢？ 视图可以具有复杂的状态。 例如，文本输入框的输入字段验证的逻辑及其取决于验证结果的文本颜色的设置，此时视图的状态不能直接保存在视图模型的字段中，也不能在IDE中进行设置。

那么，在哪里“转移”这种状态呢？模型和控制器中可以吗？

在传统的MVC中，控制器不应保存视图的状态，因此这些复杂的状态需要在Model中实现。 因此，除了域模型之外，该模型还包括部分文本输入ViewModel。

## 苹果MVC

为了适应传统MVC并解决其缺点，苹果重新构建了MVC架构，实际上是在传统的MVC的基础上构建了`Cocoa`和`CocoaTouch`框架。 在苹果的MVC下，模型与传统MVC中的模型相同，并且是主动模型（即在观察者的帮助下通知其状态的变化）。

为此，在Cocoa和CocoaTouch框架中，可以方便地使用`NSNotificationCenter`和`KVO`，而不必了解其他组件。 视图也类似于来自MVC的视图（可以是组件的层次结构）。 为了减少类的互连性，View无法直接访问Model。

![](/assets/iOSArchitecturePatterns/appleMVC.png)

用户在视图上进行也写操作，视图既能自行处理一部分视图逻辑，也能够将一部分事件转发到控制器，由控制器决定处理事务并在必要时更改模型的状态。如果模型的状态发生了更改，将通知控制器，并由控制器决定如何处理这些更改。控制器的职责还有从模型中读取数据，必要的时候会对数据进行一些转换（以便于视图使用），并对视图进行新值设定等。

### 优于传统MVC的优势

在Apple的MVC模式下，视图和模型之间不再存在直接的连接，视图的状态和数据表示的处理逻辑也在控制器中，在当前情况下，这种职责分工更为合适。

这种模式的缺点是Controller包含View状态的一部分和几乎所有View逻辑，而且由于Controller还充当View和Model之间的中介者，因此它成为应用程序逻辑适应的一个非常着重的地方。 实际上，UIViweController类变得过于庞大。 通常，由于Controller和视图之间的紧密关系，它们被视为`表示层`的组成部分。

![](/assets/iOSArchitecturePatterns/betterClassicMVC.png)


> **查看逻辑** : 一种与小部件层次管理，从一个场景到另一个场景的动画过渡，显示对话等相关的逻辑。
> 
> **表示逻辑** : 与将域模型转换为可在View上显示的模型以及处理View中需要操纵域模型的事件相关的逻辑。 
> 
> **域逻辑** : 在具有模型对象的模型级别上运行的基本逻辑。 域逻辑因此可以在另一个应用程序中重用。
> 
> **应用程序逻辑** :特定应用程序中固有的逻辑。 这与域逻辑不同，它不能重复使用，因为它是特定于特定应用程序的并且是唯一的。

## MVP

MVP（Model、View、Presenter）是MVC模式的进一步发展。 Controller由Presenter代替。 Presenter，与经典MVC中的Controller不同：

* 保存视图的状态；
* 更改视图的状态；
* 处理视图的事件；
* 将域模型转换为ViewModel。 
  
Presenter与经典MVC中的Controller也有类似之处：

* 拥有模型；
* 响应外部事件（通过调用适当的方法）更改模型的状态；
* 可能包含应用逻辑。 

MVP诞生于上世纪90年代初期的IBM。 与MVC一样，由于对其模式的不同解释，因此出现了多个版本。 马丁·福勒（Martin Fowler）定义了MVP的以下变化：

* 演示模型
* 监督控制器
* 被动视图 

它们都是相似的，但主要取决于View和Presenter之间的连接以及View的更新顺序。 小部件的层次结构通常扮演视图的角色。 MVP中的模型与MVC中的模型没有什么不同。

### 监督控制器

与MVC最接近的模式。 组件之间的相互作用如下图所示。

![](/assets/iOSArchitecturePatterns/mvp.png)

监督控制器视图：

* 实现视图逻辑；
* 将事件转发给演示者；
* 与经典MVC中一样，观察模型（在数据绑定的帮助下或实现观察者模式）；
* 不会直接更改模型的状态；
* 可能需要从Presenter请求数据或读取模型。
  
Presenter处理View的事件并更改Model的状态（通过调用适当的方法）。 与经典MVC中的Controller不同，如果无法借助数据绑定或Observer在Model与View之间建立连接的话，Presenter会保持并更改View的状态。

监督控制器的好处在于，视图状态现在位于Presenter中（而不是在Model中）。 Presenter处理演示逻辑，因此View和Model变得“更薄”。 缺点是View严重依赖Model和Presenter，这极大地使单元测试复杂化。

### 展示模型

移除了监督控制器缺点的MVP，该结构进一步开发了视图与模型之间的连接。 组件之间的交互方案是：

![](/assets/iOSArchitecturePatterns/mvp2.png)

**View：**

* 负责视图逻辑；
* 将所有事件重定向到Presenter； 

与经典的MVC和Supervision Controller不同，View无法直接访问模型。 

**Presenter：**

* 将视图的状态移动到单独的Presentation Model中，作为Presenter的一部分；
* 交互并提供与域模型的接口（即，视图的外观）；
* 观察模型状态的变化；
* 提供一个公共接口，View可以使用该接口与Presenter进行交互。 

### 该方案的工作原理如下：

视图中有一个事件，View可以尝试自行处理它，并向Presenter请求数据。 如果View无法处理该事件，它将把该事件委托给Presenter，Presenter决定如何处理该事件。 如有必要，Presenter可以更改模型的状态。 该模型将其状态更改反向通知给Presenter，Presenter读取模型的新值，如有必要，对它们执行附加逻辑并更新视图。

该模型相对于Supervision Controller的优势在于，视图与模型没有任何关系，这有利于单元测试。 缺点包括需要创建其他接口（至少对于View和Presenter而言）以及在View中进行更新的逻辑，这并不能大大简化测试。

### Humble View

Humble View和Presentation模型之间的区别在于视图及其状态如何更新。 视图变为被动，MVP的先前版本没有对View施加限制，它可能会向Presenter询​​问一些数据。 在这种情况下，被动视图受到限制，它不再向Presenter询问任何数据。

视图状态的任何更改均由Presenter执行。 视图不知道Presenter或Model的存在。 View的无源性最多可以简化单元测试。 与每种架构模式一样，组件之间的关系也有很多问题。 最常见的：

* 谁拥有MVP中的View和Presenter？

视图通常具有对Presenter的强烈引用。 反过来，Presenter对模型有很强的引用，而对View则无能为力。 与经典MVC中一样，该模型对View和Presenter一无所知。

* 谁创建View和Presenter？
  
可以认为，视图是由Presenter创建的。 但是，Presenter需要一个模型，即创建Presenter的视图必须通过模型进行配置，并且在此之后，她知道模型的存在。 此顺序不适合我们，因为我们正在尝试使组件之间的连接性达到最小（以实现更轻松的测试和更大的灵活性）。

因此，如果下一个View Presenter是由另一个Presenter创建的，或者是在单独的Router类中创建的，则更好（后者也可能参与下一个View的配置和创建）。 但是，没有明确的规则。

## iOS MVP

经过一些理论，我们可以进行实际的发展。 一个典型的iOS应用程序是围绕一个中央UIViewController类构建的，该类承担着许多责任，因此放置UI逻辑和应用程序逻辑的一部分是最有吸引力的地方。 但是，我们在上面提到，由于View和Controller之间的紧密结合（在iOS UIViewController和UIView的上下文中），将它们视为View很方便。

![](/assets/iOSArchitecturePatterns/iosmvp.png)

例如，让我们考虑一个包含两个场景的简单应用程序。 它允许您使用REST服务 http://random.cat/meow 从Internet上加载猫的随机照片（“加载猫场景”） ，在猫的图片上应用内置照片滤镜，然后保存编辑后的照片（“编辑猫”现场）。

您可以在此处下载示例应用程序： https : //github.com/thinkmobiles/CatApp_MVP_Sample

加载照片时，“加载猫场景视图”会显示活动指示器，实际加载的照片和图片的URL。 演示者将借助“最小”界面LoadCatViewProtocol与“加载猫场景视图”进行交互。

```swift
protocol LoadCatViewProtocol: View {
    
    func updateLoadingState(_ loadingState: Bool)
    func updateTitle(_ imageTitle: String?)
    func updateImage(_ image: Data?)
}
 
class LoadCatPresenter: LoadCatPresenterProtocol {
    
    weak var view: LoadCatViewProtocol!
    private var isLoading: Bool
    private var image: Data?
    private var imageTitle: String?
    
    func installView(_ view: View) {
        self.view = view as! LoadCatViewProtocol
    }
 
    func updateUI() {
        view.updateLoadingState(isLoading)
        view.updateTitle(imageTitle)
        view.updateImage(image)
    }
 
   func load() {
        
        guard !isLoading else { return }
        
        isLoading = true
        image = nil
        imageTitle = nil
        
        updateUI()
        loadCat()
    }
. . . 
}
```

加载猫场景允许您开始加载和取消它，还可以转到下一个场景进行图像编辑。 这些事件由用户启动，并且View只是将它们重定向到Presenter，调用其方法。 视图通过协议LoadCatPresenterProtocol与Presenter进行交互。

```swift
protocol LoadCatPresenterProtocol: Presenter {
    
    func load()
    func cancel()
    func updateUI()
    func edit()
    
    var catProvider: CatProvider! { get set }
}
 
class LoadCatViewController: UIViewController, LoadCatViewProtocol {
    
    var presenter: LoadCatPresenterProtocol!
 
    @IBAction func actLoad(_ sender: UIBarButtonItem) {
        presenter.load()
    }
    
    @IBAction func actCancel(_ sender: UIBarButtonItem) {
        presenter.cancel()
    }
 
	func setPresenter(_ presenter: Presenter) {
        self.presenter = presenter as! LoadCatPresenterProtocol
    }
. . . 
}
```

在我们的测试项目中，无需Router类即可过渡到下一个场景。

* 要将负载猫场景切换到编辑猫场景，您需要按编辑。
* LoadCatViewController将此事件重定向到LoadCatPresenter。
* LoadCatViewController不知道此事件会启动转换。
* LoadCatPresenter创建EditCatPresenter并使用必要的模型对其进行配置。
* 要显示下一个场景，LoadCatPresenter调用LoadCatViewController showEditScene的方法并在此处传递EditCatPresenter。
* LoadCatViewController创建下一个视图，将其与接收的Presenter连接并显示。
 
```swift
protocol LoadCatViewProtocol: View {

    func showEditScene(withPresenter presenter: Presenter)
}
 
class LoadCatViewController: UIViewController, LoadCatViewProtocol {
	
	@IBAction func actEdit(_ sender: UIBarButtonItem) {
        loadButton.isEnabled = false
        editCat()
    }
 
	func showEditScene(withPresenter presenter: Presenter) {
        let nextViewController = storyboard!.instantiateViewController(withIdentifier: Constants.editCatViewControllerStoryboardId) as! View
        presenter.installView(nextViewController)
        nextViewController.setPresenter(presenter)
        present(nextViewController as! UIViewController, animated: true, completion: nil)
    }
. . .
}
```
 
如果是第一个场景，则可以按照Apple的所有原则在UIApplicationDelegate中执行此配置。

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
 
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
 
        let view = window?.rootViewController as! LoadCatViewProtocol
        let presenter = LoadCatPresenter()
        presenter.catProvider = catProvider
        presenter.installView(view)
        view.setPresenter(presenter)
        
        return true
    }    
}
```

## MVVM

尽管MVP具有很多优点，但由于IDE开发和框架，它不适合自动化应用程序开发，需要“手动”工作。 下一个模式应该可以解决这些问题。 MVVM（Model、View、ViewModel）由Microsoft Ken Cooper和Ted Peters的工程师开发，并由John Gossman在2005年的博客中宣布。

该模式的目的是将用户界面与开发以及业务逻辑开发分开，并使用WPF和Silverlight平台的主要功能来促进应用程序测试。 尽管专业化模式是针对Microsoft技术构想的，但可以在Cocoa / CocoaTouch框架中使用。

![](/assets/iOSArchitecturePatterns/mvvm.png)

MVVM源自MVC模式，由以下3个组件组成：模型，视图，视图模型。 模型与MVP和MVC中的模型不同：

* 它是一个领域模型；
* 包括数据，业务逻辑和验证逻辑；
* 不依赖于其他组件（ View 和 ViewModel ）。 

**View：**

* 确定用户界面（如MVP，Apple MVC）的结构，位置和外观；
* 具有View的逻辑：动画、View与子View的操作之间的过渡等；
* 保持对ViewModel的强烈引用，但对Model一无所知；
* 监视ViewModel并使用数据绑定或直接引用它进行通信。 
  
为了避免View与ViewModel之间的牢固关系，需要创建一个接口，View将通过该接口与ViewModel进行通信。 ViewModel是视图和模型之间的中介者，并负责表示逻辑的处理。

**ViewModel：**

* 保持View的状态；
* 了解模型并可以更改其状态（适当类的调用方法）；
* 将模型中的数据转换为对视图更方便的格式；
* 验证来自视图的数据；
* 不了解View，只能通过数据绑定机制与View交互。
  
在Cocoa中有其自己的数据绑定机制，但在CocoaTouch中则没有。 我们只能用KVO来做，但是这个东西不方便使用，只允许您实现单边绑定。 反过来，数据绑定使实现MVVM固有的全部潜力成为可能，并总体上促进了开发。 因此，应该使用一些提供与CocoaTouch的数据绑定或响应式编程的第三方库。

MVVM和MVP中的UIViewController被视为View的一部分。

![](/assets/iOSArchitecturePatterns/mvvmcocoa.png)

从苹果的MVC到MVVM的过渡过程中出现了一个重要的问题：如何实现导航？ 如上所述，视图直接执行到其他视图的过渡。 因此，有两种方法可以进行过渡：

* 最简单的一种是从View启动过渡时。 在这种情况下，当前场景的ViewModel会创建下一个场景的ViewModel（如果需要，可以通过模型对其进行配置）。 然后，View创建下一个场景的View，将新的ViewModel传递给它，然后执行过渡。 
* 过渡从ViewModel启动。 由于ViewModel对View一无所知，因此无法进行过渡。 在这种情况下，需要一个特殊的组件-路由器-它知道视图的层次结构以及如何进行转换。 ViewModel可以将下一场景的ViewModel或模型传递给路由器。 路由器处理其他所有事务。
   
因此，MVVM和MVP（低视角）在Presentation层（在MVP中由Presenter呈现，在MVVM中由ViewModel呈现）差异很大。 MVVM优于MVP（Humble View）的优点是Presentation层完全独立于View（意味着更容易测试）和DataBinding的使用。 总之，它成为在现代IDE中使用的更具吸引力的候选者，并减少了将View与ViewModel同步的代码量。

MVVM的缺点主要在于数据绑定机制，因为在某些情况下，它可能需要大量的内存资源，并且也是内存泄漏出现的薄弱环节。 接下来，我们将考虑上一节中描述的应用程序示例，但使用MVVM模式。 您可以在此处下载示例应用程序： https : //github.com/thinkmobiles/CatApp_MVVM_Sample

应用程序和模型（Cat，CatProvider）的用户界面相同。 它们仅在表示逻辑上有所不同，这将是主要重点。 View组件由LoadCatViewController和EditCatViewController呈现。 LoadCatViewController通过以下接口与LoadCatViewModel进行交互：

```swift
protocol CatViewModelProtocol {
 
    var isLoading: Observable { get }
    var isEditable: Observable { get }
    var title: Observable<String?> { get }
    var imageData: Observable<Data?> { get }
 
    var editCatViewModel: EditCatViewModelProtocol? { get }
 
    func loadNextCat()
    func cancelCurrentDownloading()
}
```

LoadCatViewModel包含一组用于定义LoadCatViewController的状态的功能，以及一组与用户可以执行的操作相对应的方法。 对于数据绑定机制，我们使用 Bond 。 由于Load Cat是初始场景，因此很明显，它的配置是在AppDelegate中执行的：

```swift
final class AppDelegate: UIResponder, UIApplicationDelegate {
 
    var window: UIWindow?
 
    lazy var catProvider = CatProvider()
 
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        let catViewModel = CatViewModel(catProvider: catProvider)
        (window?.rootViewController as? CatViewController)?.viewModel = catViewModel
        return true
    }
}
```

与MVP一样，配置Edit Cat场景分别在View和ViewModel中进行。

```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let editCatViewController = segue.destination as? EditCatViewController {
            editCatViewController.viewModel = viewModel.editCatViewModel
        }
    }
```

* 首先，使用Segue机制的LoadCatViewController创建EditCatViewController。
* 然后，在prepareForSegue方法中，LoadCatViewController询问LoadCatViewModel下一个场景的已配置ViewModel，即包含当前Cat模型的EditCatViewModel。
* 此外，我们将此EditCatViewModel传递给EditCatViewController。
* 单元测试是ViewModel和Model中的应用程序测试。 在测试项目中，您将找到单元测试的示例。

## VIPER

在上面描述的架构中，如果尝试将体系结构划分为多个层，则使用Presenter或View Model可能会遇到困难。 它们属于哪一层？ 这个问题没有明确的答案：您可以为Presenter引入一个单独的Presentation层，或者它可以属于Application Logic。 MVVM也一样。 这种歧义造成了另一个问题。

将应用逻辑与域模型逻辑分开是非常困难的。 因此，通常没有分隔并且位于同一层。 此外，Presenter中存在应用程序逻辑有时会使很难测试不同的用例。 以前的体系结构中的另一个问题是组装和导航。 在数十个场景的大型项目中，很明显，这是由单独的模块路由器负责的。

2012年 发表 了一篇非凡的文章 。 清洁建筑以及有关该主题的几篇演讲。 后来，在Mutual Mobile中，我们为iOS做了一些修改，并进入了VIPER的新模式。 它 是View，Interactor，Presenter，Entity，Router的缩写，它们是构成应用程序的基本组件。 在下面查看他们如何互动。

![](/assets/iOSArchitecturePatterns/VIPER.png)

**View：**

与MVP（被动视图）一样，它是来自Presenter的数据的可视化。 View通过高于UI类级别的协议与Presenter通信。 演示者不知道构成视图层次结构的特定类。 要在View和Presenter之间共享数据，可以使用单独的结构（即，没有方法可以更改其状态的类）。 只有View和Presenter知道这些类。

**Presenter：** 与MVP中的功能相同，不同之处在于它不应包含应用程序逻辑。 我们主要让Presenter参与数据转换。

**Interactor：** 这些对象封装了应用程序的单独用例（我们将其称为应用逻辑）。 交互器与演示者和模型一起使用。 Interactor永远不会将属于模型层的对象类传递给Presenter。 因此，演示者不依赖于模型。 而且，他不知道该模型的存在。

**Model：** 与以前的模式相同。 对于方向模型，只有交互器起作用。 该模型不知道其他组件的存在。 模型层可能包含各种管理器（用于创建或保留实体）和封装数据处理算法的对象。

**Entity：** 实体是仅包含数据且不包含其处理方法的PONSO（普通的NSObject）对象（例如，其所有属性均为只读，并且NSManagerObject类的对象不能脱离模型层的边界）。

**Routing：** 线框和演示者负责VIPER中的导航。

演示者接收视图的事件并知道如何响应它们。 但是Presenter对View的层次结构一无所知，并且包含View Logic（场景之间的动画切换– View Logic示例），并且无法在场景之间切换。

在这里，它将需要Wireframe（一个包含对UIWindow的引用的对象），可以创建View / UIViewController并知道如何将它们放入View层次结构中。 同样，线框是诸如场景之间的自定义过渡之类的事务处理的理想位置。 例如，让我们考虑一个测试项目的VIPER版本，上面已针对MVP进行了描述。

您可以在此处下载示例代码： https : //github.com/thinkmobiles/CatApp_VIPER_Sample

在项目的MVP和VIPER版本中比较LoadCatView的协议。


<table>
<tbody>
<tr>
<td><strong>MVP</strong></td>
<td><strong>VIPER</strong></td>
</tr>
<tr>
<td><span style="font-weight: 400;">protocol</span><span style="font-weight: 400;"> LoadCatViewProtocol: </span><span style="font-weight: 400;">View</span><span style="font-weight: 400;"> {</span><p></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateLoadingState(</span><span style="font-weight: 400;">_</span><span style="font-weight: 400;"> loadingState: </span><span style="font-weight: 400;">Bool</span><span style="font-weight: 400;">)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateTitle(</span><span style="font-weight: 400;">_</span><span style="font-weight: 400;"> imageTitle: </span><span style="font-weight: 400;">String</span><span style="font-weight: 400;">?)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateImage(</span><span style="font-weight: 400;">_</span><span style="font-weight: 400;"> image: </span><span style="font-weight: 400;">Data</span><span style="font-weight: 400;">?)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> showEditScene(withPresenter presenter: </span><span style="font-weight: 400;">Presenter</span><span style="font-weight: 400;">)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> finishedEdit()</span></p>
<p><span style="font-weight: 400;">}</span></p></td>
<td><span style="font-weight: 400;">protocol</span><span style="font-weight: 400;"> LoadCatViewProtocol: </span><span style="font-weight: 400;">View</span><span style="font-weight: 400;"> {</span><p></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateLoadingState(</span><span style="font-weight: 400;">_</span><span style="font-weight: 400;"> loadingState: </span><span style="font-weight: 400;">Bool</span><span style="font-weight: 400;">)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateTitle(</span><span style="font-weight: 400;">_</span><span style="font-weight: 400;"> title: </span><span style="font-weight: 400;">String</span><span style="font-weight: 400;">?)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateImage(</span><span style="font-weight: 400;">_</span><span style="font-weight: 400;"> image: </span><span style="font-weight: 400;">UIImage</span><span style="font-weight: 400;">?)</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> finishEditing()</span></p>
<p><span style="font-weight: 400;">}</span></p></td>
</tr>
</tbody>
</table>



它们仅在方法上有所不同

```swift
func showEditScene(withPresenter presenter: Presenter) 
```

因为就VIPER而言，新场景或对话显示是线框的职责。 因此，两个项目的LoadCatViewProtocol实现几乎相同。 比较MVP和VIPER项目的LoadCatPresenter。

<table>
<tbody>
<tr>
<td><strong>MVP</strong></td>
<td><strong>VIPER</strong></td>
</tr>
<tr>
<td><span style="font-weight: 400;">protocol</span><span style="font-weight: 400;"> LoadCatPresenterProtocol: </span><span style="font-weight: 400;">Presenter</span><span style="font-weight: 400;"> {</span><p></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> load()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> cancel()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateUI()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> edit()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">ar</span><span style="font-weight: 400;"> catProvider: </span><span style="font-weight: 400;">CatProvider</span><span style="font-weight: 400;">! { </span><span style="font-weight: 400;">get</span> <span style="font-weight: 400;">set</span><span style="font-weight: 400;"> }</span></p>
<p><span style="font-weight: 400;">}</span></p></td>
<td><span style="font-weight: 400;">protocol</span><span style="font-weight: 400;"> LoadCatPresenterProtocol: </span><span style="font-weight: 400;">Presenter</span><span style="font-weight: 400;"> {</span><p></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> load()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> cancel()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> updateUI()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">func</span><span style="font-weight: 400;"> edit()</span></p>
<p><span style="font-weight: 400;"> &nbsp;&nbsp;&nbsp;</span><span style="font-weight: 400;">var</span><span style="font-weight: 400;"> loadCatInteractor: </span><span style="font-weight: 400;">LocadCatInteractor</span><span style="font-weight: 400;">! { </span><span style="font-weight: 400;">get</span> <span style="font-weight: 400;">set</span><span style="font-weight: 400;"> }</span></p>
<p><span style="font-weight: 400;">}</span></p></td>
</tr>
</tbody>
</table>

它们之间的区别不是很大。 该项目的MVP版本包含一个变量catProvider，该变量引用了Model层。 在项目的VIPER版本中，Presenter不必依赖于Model层。

由于通过按下按钮加载图片是一个用例（或应用程序逻辑），因此要实现功能，需要一个Interactor（可变loadCatInteractor）。 通常，交互器具有输入（演示者可以通过其与之交互的接口）。

```swift
protocol LoadCatInteractorInput { 
    func loadCat() 
    func cancelLoad() 
    } 
```

和与演示者交互的输出

```swift
 protocol LoadCatInteractorOutput { 
     func didLoadCatURL(_ catURL: NSURL?, success: Bool, cancelled: Bool) 
     func didLoadCatImage(_ image: Data?, success: Bool, cancelled: Bool) 
 } 
```

因此，通过按下按钮加载猫图片处理如下所示

```swift
class LoadCatViewController: UIViewController, LoadCatViewProtocol {
    
    var presenter: LoadCatPresenterProtocol!
    
    @IBAction func actLoad(_ sender: UIButton) {
        presenter.load()
    }
. . . 
}
 
class LoadCatPresenter: LoadCatPresenterProtocol, LoadCatInteractorOutput {
 
    var loadCatInteractor: LocadCatInteractor!
     
    public func load() {
        
        // Some code to prepare UI
        loadCatInteractor.loadCat()
    }
    
    //MARK: LoadCatInteractorOutput
    
    func didLoadCatURL(_ catURL: NSURL?, success: Bool, cancelled: Bool) {
        // Show the URL
    }
    
    func didLoadCatImage(_ image: Data?, success: Bool, cancelled: Bool) {
        // Show the image
    }
    . . . 
}
```

交互器和模型层之间的交互。 模型层由CatProvider和Cat类表示。 由于它很原始，因此对于交互器和模型之间的数据交换，我们没有创建实体类。

让我们考虑在场景之间切换。 正如我们上面提到的，在VIPER项目中，这是线框的责任。 如果下一个场景需要上一个场景的某些数据，则可以将它们传递到线框中。

```swift
class LoadCatPresenter: LoadCatPresenterProtocol, EditCatPresenterDelegate {
 
    func edit() {
        let image = UIImage(data: self.image!)
        let editCatPresenter = EditCatPresenter()
        editCatPresenter.delegate = self
        editCatPresenter.image = image!
        
        view.showEditScene(withPresenter: editCatPresenter)
    }
. . .
}
```

因此，不再将Seguey机制用于场景之间的过渡是不方便的，但是也不是拒绝使用UIStoryboard这样的便捷机制处理场景的理由。 这里的场景将没有Seguey。

一个普通的VIPER项目包含许多您需要配置的模块。 对于我们的简单示例，在应用程序启动时使用单独的Dependencies类就足够了。 但是，在复杂的项目中，更容易使用其他解决方案或库。

**测试:** VIPER项目的测试与MVP相似，不同之处在于将应用逻辑交付到单独的类–交互器中。 一方面，您必须编写更多用于单元测试的代码，另一方面，还需要针对单个功能测试（用户案例）使用更简单的算法。 在我们的测试项目中，您将找到所有VIPER项目的单元测试示例。

## 结论

在本文中，我们研究了可用于开发iOS应用程序的体系结构模式的演变。 进化链中的每个模式都改进了前一个模式。 明确了组件之间的界限及其职责（如有必要，引入了新的层或组件），这有助于开发和支持。

## 参考资料

* [iOS architecture patterns: A guide for developers](https://thinkmobiles.com/blog/ios-architecture-patterns/)
* [iOS 架构模式 - 简述 MVC, MVP, MVVM 和 VIPER](https://blog.coding.net/blog/ios-architecture-patterns)
* [iOS的MVP设计模式](https://www.jianshu.com/p/61a656431685)