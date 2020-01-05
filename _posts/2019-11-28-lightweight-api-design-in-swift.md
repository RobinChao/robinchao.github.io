---
layout: post
author: Robin
title: Swift语言中的轻量级API设计
tags: 开发知识 iOS
categories:
- 开发知识
cover: '/assets/lightweight-api/cover.jpg'
---

Swift语言自诞生以来，总是或多或少受到人们的非议，新生的编程语言难免有些不够尽善尽美，但是哪种编程语言是尽善尽美的呢？OC语言算得上是一种古老的面向对象语言了，发展至今，其版本仍处于2.0，但是Apple为了让其看起来强大一点，增加了很多特性，例如Block、instancetype等等，但是其核心的语法变化并不大。

截止目前，Swift的版本已经迭代到5.*，整个ABI也已经稳定，每一次迭代更新，总是会带来一些漂亮的设计模式实践，例如在如何设计API方面，给开发者带来了舒适而强大的枚举、扩展和协议等，不仅让开发者对于函数的定义有了更清晰的认识，而且对于构建API而言，第一印象往往是轻量的，同时，仍会根据需要逐步显现出更多的功能，以及底层的复杂性。

在本篇文章里，将尝试创建一些轻量级的API，以及如何使用API组合的力量使得功能或者系统更加强大等。

## 功能和易用性之间的较量

通常，当我们设计API时，会在数据结构和函数功能的相互交互上，寻找一个相对平衡的方式，最终构建出在功能上满足需求，数据结构尽量简单的API。但是，让API过于简单，可能它们又不够灵活，无法使功能有不断发展的潜力，然而，太过复杂的设计又难免导致开发工作复杂而无章法，容易造成开发者挫败，逻辑混乱而且API也难以使用，最终可能会导致延期甚至失败。

例如，一款应用程序的主要功能是对用户选择的图像应用不同的滤镜效果。每一种滤镜的核心其实都是一组图像变换的组合，不同的变换组合形成不同的滤镜效果。假设使用`ImageFilter` 结构体作为图像滤镜的定义，如下：

```swift
struct ImageFilter {
    var name: String
    var icon: Icon
    var transforms: [ImageTransform]
}
```

`ImageTransform`是图像变换的统一入口，因为可能会由多种不同的变换，因此可以将其定义为一个`protocol`，然后由实现单独变换操作的各种变换类型所遵循：

```swift
protocol ImageTransform {
    func apply(to image: Image) throws -> Image
}

struct PortraitImageTransform: ImageTransform {
    var zoomMultiplier: Double
    
    func apply(to image: Image) throws -> Image {
        ...
    }
}

struct GrayScaleImageTransform: ImageTransform {
    var brightnessLevel: BrightnessLevel
    
    func apply(to image: Image) throws -> Image {
        ...
    }
}
```

上述设计方式的优势在于，由于每种转换都是按照自己的类型实现的，因此在使用时可以自由地让每种变换类型定义自己所需的属性和参数。例如`GrayScaleImageTransform` 接受 `BrightnessLevel`参数，以将图像转换为灰度图像。

然后，可以根据需要组合任意数量的图像变换类型，以形成不同类型的滤镜效果。例如，通过一系列的转换使得图像具有某种“戏剧性”外观的滤镜：

```swift
let dramaticFilter = ImageFilter(
    name: "Dramatic", icon: .drama, transforms: [
        PortraitImageTransform(zoomMultiplier: 2.1),
        ContrastBoostImageTransform(),
        GrayScaleImageTransform(brightnessLevel: .dark)
    ]
)
```

So far so Good.  但是回头重新审视上述API的实现，可以肯定的说，上述实现仅仅是为了功能的实现，在API的易用性方面并没有优势，那么该如何进行优化，来保证功能的同时，提高API的灵活性和易用性呢？在上述实现中，每个图像的变换都是作为单独的类型实现的，因此没有一个可以对所有变换类型一目了然的地方，使用者难以清楚该代码库都包含哪些图像变换的类型。

为了解决外部使用者无法得知软件库所支持的变换类型，假设使用**枚举**的方式代替上述方式，来观察哪种方式更能够体现API的简洁明了以及使用上的清晰易用？

```swift
enum ImageTransform {
    case protrait(_ zoomMultiplier: Double)
    case grayScale(_ brightnessLevel: BrightnessLevel)
    case contrastBoost
}
```

使用枚举的好处既能够提高代码的整洁程度和可读性，也使得API更加的灵活易用，因为在枚举的使用上，开发者可以直接使用点语法构造任意数量的转换，如下：

```swift
let dramaticFilter = ImageFilter(
    name: "Dramatic",
    icon: .drama,
    transforms: [
        .protrait(2.1),
        .contrastBoost,
        .grayScale(.dark)
    ]
)
```

截止目前，枚举都是很漂亮的一个工具，在很多情况下Swift的枚举类型都能够提供良好的解决方式，但是枚举也有其明显的弊端。

就上述例子来说，由于每个转换都需要执行截然不同的图像操作，因此在这种情况下使用枚举将迫使我们编写一个庞大的*switch*语句来处理这些操作中的每一项, 这可能会造成代码的冗长繁琐等。

## 枚举虽轻，结构体更优

幸运的事，针对上述问题，我们还有第三种选择 --- 一种目前算是两全其美的方案。相较于协议或者枚举，结构体是一个既能够定义操作类型，还能够封装给定各种操作的闭包的数据结构。例如：

```swift
struct ImageTransform {
    let closure: (Image) throws -> Image

    func apply(to image: Image) throws -> Image {
        try closure(image)
    }
}
```

> `apply(to:)` 方法在这里并不应该被外部调用，这里写出来是为了代码的美观性以及代码的向前兼容。在实际项目开发中，这里可以使用宏定义区分。

完成上述操作后，我们现在可以使用**静态工厂方法**和属性来创建我们的转换 --- 每个转换仍可以单独定义并具有自己的一组参数：

```swift
extension ImageTransform {
    static var contrastBoost: Self {
        ImageTransform { image in
            // ...
        }
    }
    
    static func portrait(_ multiplier: Double) -> Self {
        ImageTransform { image in
            // ...
        }
    }
    
    static func grayScale(_ brightness: BrightnessLevel) -> Self {
        ImageTransform { image in
            // ...
        }
    }
}
```


> 在 Swift 5.1 中，可以将**Self**用作静态工厂方法的返回类型。

上面方法的优点在于，我们回到了将ImageTransform定义为协议时所具有的灵活性和功能性，同时仍保持了与定义为枚举时的调用方式 --- 点语法一致，保证了易用性。

```swift
let dramaticFilter = ImageFilter(
    name: "Dramatic",
    icon: .drama,
    transforms: [
        .portrait(2.1),
        .contrastBoost,
        .grayScale(.dark)
    ]
)
```

点语法本身与枚举无关，但是其可以与任何静态API一起使用，这点对于开发者而言非常友好。使用点语法可以将上述的几个滤镜的创建和建模构造成静态属性，使得我们能够进一步的封装特性等。例如：

```swift
extension ImageFilter {
    static var dramatic: Self {
        ImageFilter(
            name:"Dramatic",
            icon: .drama,
            transforms: [
                .portrait(2.1),
                .contrastBoost,
                .grayScale(.dark)
            ]
        )
    }
}
```

通过上述改造，一系列复杂的任务 --- 包括图像滤镜和图像转换 -- 封装到一个API中，在使用上，可以像传值给函数一样轻松。

```swift
let filtered = image.withFilter(.dramatic)
```

上述一系列的改造可以成为为类型构造**语法糖**。不仅改善了API读取的方式，还改善了API的组织方式，由于所有的转换和滤镜现在只需要进行传单一的值即可，因此在可扩展性方面来说，能够组织多种方式，不仅使得API轻巧灵活，对于使用者来说也简洁明了。

## 可变参数与API设计

接下来我们一起看看Swift语言的另一个特性 --- 可变参数，以及可变参数如何影响API设计中的代码构建的。

假设正在开发一个使用基于形状的绘图来创建其用户界面的应用程序，并且我们已经使用了与上述类似的基于结构的方法来对每种形状进行建模，并最终将结果绘制到了`DrawingContext`中：

```swift
struct Shape {
    var drawing: (inout DrawingContext) -> Void
}
```

> 上面使用**inout**关键字来启用值类型（DrawingContext）的传递。

类似我们在上面例子中使用静态工厂方法轻松创建`ImageTransform`一样，在这里也能够将每个形状的绘图代码封装在一个完全独立的方法中，如下所示：

```swift
extension Shape {
    func square(at point: Point, sideLength: Double) -> Self {
        Shape { context in
            let origin = point.movedBy(
                x: -sideLength / 2,
                y: -sideLength / 2
            )

            context.move(to: origin)
            context.drawLine(to: origin.movedBy(x: sideLength))
            context.drawLine(to: origin.movedBy(x: sideLength, y: sideLength))
            context.drawLine(to: origin.movedBy(y: sideLength))
            context.drawLine(to: origin)
        }
    }
}
```

由于将每个形状简单地建模为一个属性值，因此绘制它们的数组变得非常容易-我们要做的就是创建一个**DrawingContext**实例，然后将其传递到每个形状的闭包中以构建最终图像：

```swift
func draw(_ shapes: [Shape]) -> Image {
    var context = DrawingContext()
    
    shapes.forEach { shape in
        context.move(to: .zero)
        shape.drawing(&context)
    }
    
    return context.makeImage()
}
```

调用上面的函数看起来也很优雅，因为我们再次可以使用点语法来大大减少执行工作所需的语法量：

```swift
let image = draw([
    .circle(at: point, radius: 10),
    .square(at: point, sideLength: 5)
])
```

但是，让我们看看是否可以使用可变参数来使事情更进一步。虽然不是Swift独有的功能，但结合Swift真正灵活的参数命名功能后，使用可变参数可以产生一些非常有趣的结果。

当参数被标记为可变参数时（通过在其类型中添加`...`后缀），我们基本上可以将任意数量的值传递给该参数 --- 编译器会自动为我们将这些值组织到一个数组中，例如这个：

```swift
func draw(_ shapes: Shape...) -> Image {
    ...
    // Within our function, 'shapes' is still an array:
    shapes.forEach { ... }
}
```

完成上述更改后，我们现在可以从对draw函数的调用中删除所有数组文字，而使它们看起来像这样：


```swift
let image = draw(.circle(at: point, radius: 10),
                 .square(at: point, sideLength: 5))
```

这看起来似乎不是很大的变化，但是尤其是在设计旨在用于创建更多更高级别值（例如我们的draw函数）的更低级别的API时，使用可变参数可以使这类API感觉更轻巧和方便。

但是，使用可变参数的一个缺点是，预先计算的值数组不能再作为单个参数传递。值得庆幸的是，在这种情况下，可以通过创建一个特殊的组形状（就像draw函数本身一样），在一组基础形状上进行迭代并绘制它们来轻松解决：

```swift
extension Shape {
    static func group(_ shapes: [Shape]) -> Self {
        Shape { context in
            shapes.forEach { shape in
                context.move(to: .zero)
                shape.drawing(&context)
            }
        }
    }
}
```

完成上述操作后，我们现在可以再次轻松地将一组预先计算的Shape值传递给我们的draw函数，如下所示：

```swift
let shapes: [Shape] = loadShapes()
let image = draw(.group(shapes))
```

不过，真正酷的是，上述组API不仅使我们能够构造形状数组，而且还使我们能够更轻松地将多个形状组合到更高级的组件中。例如，这是我们如何使用一组组合形状来表示整个图形（例如徽标）的方法：

```swift
extension Shape {
    static func logo(withSize size: Size) -> Self {
        .group([
            .rectangle(at: size.centerPoint, size: size),
            .text("The Drawing Company", fittingInto: size),
            ...
        ])
    }
}
```

由于上述徽标与其他徽标一样都是Shape，因此只需调用一次draw方法就可以轻松绘制它，并使用与之前相同的优雅点语法：


```swift
let logo = draw(.logo(withSize: size))
```

有趣的是，尽管我们最初的目标可能是使我们的API更轻量级，但这样做也使它的可组合性和灵活性也得到了提高。


## 总结

我们向“ API设计者的工具箱”添加的工具越多，我们越有可能能够设计出在功能，灵活性和易用性之间达到适当平衡的API。 使API尽可能轻巧可能不是我们的最终目标，但是通过尽可能减少API的数量，我们也经常发现如何使它们变得更强大-通过使我们创建类型的方式更灵活，以及使他们组成。所有这些都可以帮助我们在简单性与功能之间实现完美的平衡。

> 原文： Lightweight API design in Swift
> 链接：https://www.swiftbysundell.com/articles/lightweight-api-design-in-swift