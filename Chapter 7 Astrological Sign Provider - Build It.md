# Chapter 7 星座提供者：实现

构建一个 `AstrologicalSignProvider` 的过程相当直接了当。我们将会构造最基本的元素，并向其中为每一个星座填充内容。然后我会给你一份最终文件的副本，因为构造剩余的 11 个星座仅仅要做的就是复制粘贴的工作（既然有一个做好的文件在那，就没有必要再做了）。

你将会做下面的这些事情：

1. 构建一个新的 `NSObject` 的子类 `AstrologicalSignProvider`
2. 构建一个结构体 `AstrologicalSign`，用来保存一个星座的数据
3. 构建一个 `typealias` 用来表示一个可以创建 `AstrologicalSign` 的方法
4. 构建一系列字符串之间的映射，以及每个星座的类型别名
5. 构建一个 `get` 方法，使得你可以通过请求一个星座的名字来抓取数据
6. 为每个星座构建一个方法用来返回星座数据

> 从现在开始，我们将会编辑 `AstrologicalSignProvider.swift`。


## 1. 构建一个结构体

在类声明之上，构建一个叫 `AstrologicalSign` 的结构体，它包含四个变量，分别叫 `shape`、`big`、`small` 和 `lines`，像这样：

```swift
struct AstrologicalSign {
    var shape : Shape!
    var big : [Point]!
    var small : [Point]!
    var lines : [[Point]]!
}

class AstrologicalSignProvider : NSObject {...
```

我们可以访问这个结构体的 `Shape`（就是图标），两组坐标数组用来标记大星星和小星星的中心位置，以及一个存放坐标数组的数组使得我们知道在哪些星星之间要连线。

紧接着，构建一个类型别名叫它 `AstrologicalSignFunction`，用来定义一种能够返回图标结构体的方法。将它添加到 `AstrologicalSign` 上方，像这样：

```swift
typealias AstrologicalSignFunction = () -> AstrologicalSign

struct AstrologicalSign {...
```

最后，使用 `let` 关键字，添加一个固定字符串数组作为这个类的一个常量。这个数组代表十二星座的顺序。

```swift
let order = ["pisces", "aries", "taurus", "gemini", "cancer", "leo", "virgo", "libra", "scorpio", "sagittarius", "capricorn", "aquarius"]
```

## 2. 单例

我们将从不同的类引用 provider，但 provider 本身却不会改变。我们不会在每个不同的类里实例化一个本地的 provider，而是将创建一个 `sharedInstance` —— 称之为单例 —— 能保证只被创建一次。我们可以从我们项目中的任何对象中获得这个实例。

将这个属性添加到类中：

```swift
static let sharedInstance = AstrologicalSignProvider()
```

这时候你的文件应该看起来像：

```swift
import Foundation

typealias AstrologicalSignFunction = () -> AstrologicalSign

struct AstrologicalSign {
    var shape : Shape!
    var big : [Point]!
    var small : [Point]!
    var lines : [[Point]]!
}

class AstrologicalSignProvider : NSObject {
    static let sharedInstance = AstrologicalSignProvider()
    
    let order = ["pisces", "aries", "taurus", "gemini", "cancer", "leo", "virgo", "libra", "scorpio", "sagittarius", "capricorn", "aquarius"]

    override init() {
        super.init()
    }
}
```

## 3. 伪造所有的星座方法

接下来，我们将为星座创建方法，但不具体实现方法。我们先将它们伪造出来，这样我们就能够先完成剩下的类再来完它们。这样会更容易…

相信我。

为十二星座的每个图标创建一个像下面这样的方法：

```swift
func signName() -> AstrologicalSign {
   let sign = AstrologicalSign()
   return sign
}
```

然后替换星座的名字。白羊座对应的方法像这样：

```swift
func aries() -> AstrologicalSign {
    ...
}
```

到现在，你的类应该看起来像这样：

```swift
import C4

typealias AstrologicalSignFunction = () -> AstrologicalSign

struct AstrologicalSign {
   var shape = Shape()
   var big = [Point]()
   var small = [Point]()
   var lines = [[Point]]()
}

class AstrologicalSignProvider : NSObject {
   let order = ["pisces", "aries", "taurus", "gemini", "cancer", "leo", "virgo", "libra", "scorpio", "sagittarius", "capricorn", "aquarius"]

   func taurus() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func aries() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func gemini() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func cancer() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func leo() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func virgo() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func libra() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func pisces() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func aquarius() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func sagittarius() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func capricorn() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }

   func scorpio() -> AstrologicalSign {
       let sign = AstrologicalSign()
       return sign
   }
}
```

## 4. 构造映射关系

接下来我们要做的是创建一组映射关系，将使我们仅通过请求一个星座的名字来获得这个星座的图标。在这步中，我们开始使用我们前几个步骤前构建的类型别名（`typealias`）。

在 `let order = [...]` 属性后添加以下代码：

```swift
internal var mappings = [String : AstrologicalSignFunction]()
```

Swift 有一个很好的特性，它允许我们把函数放入数组和字典，不久之后我们会利用到这点。`internal` 关键字意味着变量不能从类外部访问（即只有内部可用）。

### 4.1. 初始化 —— init() 

下一步是初始化映射关系。所以，修改类的 init 方法和设置键和变量的映射。你所有需要做的就是为每个星座增加它的名称，并将其与它的方法联系起来，像这样：

```swift
override init() {
   super.init()
   mappings = [
   "pisces":pisces,
   "aries":aries,
   "taurus":taurus,
   "gemini":gemini,
   "cancer":cancer,
   "leo":leo,
   "virgo":virgo,
   "libra":libra,
   "scorpio":scorpio,
   "sagittarius":sagittarius,
   "capricorn":capricorn,
   "aquarius":aquarius
   ]
}
```

> 你可以在 mappings 变量后面添加这个方法。

### 4.2. 一点点解释

所以我们已经做了一些可能没有多大意义的事情，因为这个类实际上不做任何事。然而我们也完成了许多事情。

这是我们已经完成的：

1. 我们已经创建了一个 `AstrologicalSign` 的结构体用于表示星座（但还没有真正使用它们）
2. 我们已经创建了一个类型别名代表函数
3. 我们创建了十二个函数用于返回星座结构体
4. 我们已经创建了一个映射，将每一个词典基于字符串的函数名称（不久后将使用）
5. 我们创建了一个初始化函数

现在开始，我们将创建一个方法，能够通过名字直接获取一个星座数据。我们会跟进直到完成所有空的星座方法。

## 5. 获得 —— Get()

获得星座的数据只需要将星座的名字作为参数即可。为什么这么做呢？因为这让我们的项目的剩余部分更加整洁，也便于我们分阅读代码，了解发生了什么。

`get` 方法很简单，它看起来像这样：

```swift
func get(sign: String) -> AstrologicalSign? {
    let function = mappings[sign.lowercaseString]
    return function!()
}
```

这个方法之所以设计得精巧，就在于它只基于星座的名称检索星座，再返回方法的返回值而不是返回一个方法（即 `()` 所做的就是将方法执行，而 `!` 能够获得返回闭包结果的原始值）。

所以从现在开始，在其他类中调用这个函数就像这样：

```swift
signProvider.get("aries")
```

上述会返回一个白羊座星座的 `AstrologicalSign` 结构体。 

精妙！

## 6. 补全方法

让我们补全第一个方法，让剩下的代码成为复制粘贴的简单活儿。

首先，让我们给白羊座的 `aries()` 方法添加一个路径。

```swift
let bezier = Path()
```

尽管这是一个常量，但我们还是可以对其添加点、曲线和线条。

要创建白羊座图标，在创建了贝塞尔曲线以后添加以下内容：

```swift
bezier.moveToPoint(Point(2.8, 15.5))
bezier.addCurveToPoint(Point(0, 9), control1:Point(1.1, 13.9), control2:Point(0, 11.6))
bezier.addCurveToPoint(Point(9, 0), control1:Point(0, 4), control2:Point(4, 0))
bezier.addCurveToPoint(Point(18, 9), control1:Point(14, 0), control2:Point(18, 4))
bezier.addLineToPoint(Point(18, 28.9))
        
bezier.moveToPoint(Point(18, 28.9))
bezier.addLineToPoint(Point(18, 9))
bezier.addCurveToPoint(Point(27, 0), control1:Point(18, 4), control2:Point(22, 0))
bezier.addCurveToPoint(Point(36, 9), control1:Point(32, 0), control2:Point(36, 4))
bezier.addCurveToPoint(Point(33.2, 15.5), control1:Point(36, 11.6), control2:Point(34.9, 13.9))
```

这些方法调用构造白羊座的形状变成我们刚刚创建的贝塞尔曲线。

接下来，我们将创建星星（大星星和小星星）。添加以下内容:

```swift
let big = [
   Point(58.0,-57.7),
   Point( )]
let small = [
   Point(-134.5,-95.2),
   Point(137.0,11.3)]
```

接下来，我们将要添加一个能够容纳数组的数组 :)...

```swift
let lines = [
   [big[0],small[0]],
   [big[1],big[0]],
   [big[1],small[1]]]
```

这个数组存放了前两个数组中的点集，第一个入口是：

```swift
[Point(58.0,-57.7), Point(-134.5,-95.2)]
```

我们本可以使用元组，但是由于创建一条线（`Line`）需要传递一系列的点。并且，基于这种结构我们现在可以通过以下方式来创建线条：

```swift
let l = Line(lines[0])
```

现在，来结束这个方法我们需要填充我们将会返回的星座信息。因为我们已经创建了方法，我们所有需要做的就是添加如下返回语句之前的内容：

```swift
sign.shape = Shape(bezier)
sign.big = big
sign.small = small
sign.lines = lines
```

到现在我们已经添加了许多行到 `aries()` 方法中了，它应该看起来像这样：

```swift
func aries() -> AstrologicalSign {
    let bezier = Path()
    
    bezier.moveToPoint(Point(2.8, 15.5))
    bezier.addCurveToPoint(Point(0, 9), control1:Point(1.1, 13.9), control2:Point(0, 11.6))
    bezier.addCurveToPoint(Point(9, 0), control1:Point(0, 4), control2:Point(4, 0))
    bezier.addCurveToPoint(Point(18, 9), control1:Point(14, 0), control2:Point(18, 4))
    bezier.addLineToPoint(Point(18, 28.9))
    
    bezier.moveToPoint(Point(18, 28.9))
    bezier.addLineToPoint(Point(18, 9))
    bezier.addCurveToPoint(Point(27, 0), control1:Point(18, 4), control2:Point(22, 0))
    bezier.addCurveToPoint(Point(36, 9), control1:Point(32, 0), control2:Point(36, 4))
    bezier.addCurveToPoint(Point(33.2, 15.5), control1:Point(36, 11.6), control2:Point(34.9, 13.9))
    
    var sign = AstrologicalSign()
    sign.shape = Shape(bezier)
    
    let big = [
        Point(58.0,-57.7),
        Point(125.5,-17.7)]
    sign.big = big
    
    let small = [
        Point(-134.5,-95.2),
        Point(137.0,11.3)]
    sign.small = small
    
    let lines = [
        [big[0],small[0]],
        [big[1],big[0]],
        [big[1],small[1]]]
    sign.lines = lines
    
    return sign
｝
```

## 7. 最后一点

你并不需要像我一样从 Numbers 中复制粘贴了！直接下载这个文件吧：

[AstrologicalSignProvider.swift](https://gist.github.com/C4Framework/37d4f98f308f783b102e)

然后，回到您项目的工作区测试它并且覆盖 `setup()` 方法，看起来像这样：

```swift
override func setup() {
   let provider = AstrologicalSignProvider()
   let aries = provider.get("aries")
   print(aries)
}
```

然后在控制台上会打印以下内容：

```
Optional(COSMOS.AstrologicalSign(shape: <COSMOS.Shape: 0x7fb1c0e0a280>, big: [{58.0, -57.7}, {0.0, 0.0}], small: [{-134.5, -95.2}, {137.0, 11.3}], lines: [[{58.0, -57.7}, {-134.5, -95.2}], [{0.0, 0.0}, {58.0, -57.7}], [{0.0, 0.0}, {137.0, 11.3}]]))
```

啊，是的。这真是太好了。

记得删除 `WorkSpace` 中的代码，它只是用于测试。
