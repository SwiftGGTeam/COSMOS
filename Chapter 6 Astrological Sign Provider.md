# Chapter 6 星座提供者

我们将会创建一个持有星座的所有数据（位置，形状等）的对象。它还将定义一个 `AstrologicalSign` 结构，将为一个星座保存它图标的形状、小星星和大星星的位置，以及让我们用来描边的每根连线的位置的信息。

但是，在我们开始之前，我们需要看设计文档来大致了解一下哪些是我们需要构建的。

## 1. 逻辑

菜单图标和背景都引用每个十二星座的相关图标。

![](http://www.c4ios.com/images/cosmos/6/01.png)

从这些设计规格里我们可以得出两件事。并且通过和 Jake 的交谈我得出了第三点。

1. 星座会被用在两个不同的对象上（如菜单和背景）
2. 星座要适应不同大小比例的渲染
3. 图标能够从一个点变换成完整的形状

基于这三点我得到了下面的结论：

1. 我不想将相同的形状数据硬编码到两个独立的对象，所以当我需要形状数据的时候需要一个单独的对象来提供
2. 我不能使用图片，因为我需要在它的形状上做些动画
3. 形状应该用贝塞尔曲线（即由一系列的直线和曲线）来描绘，这样我可以很容易地实现缩放和加载动画

这就是为什么我们要构建一个 `AstrologicalSignProvider`。

但是，在我们开始之前，需要通过编码来实现星座的形状。

### 1.1. 图标形状

有两种获得星座形状的方法。我们大可以添加一个图像到一个新的 C4 项目，然后往项目里痛苦地添加点集和曲线，猜测控制点和线的位置直到我们从头构建每个星座。然而，在所有不同的图标里大概有 200 个不同的点...猜测所有这些点和线的位置可能要花上一个星期的功夫。<sup><a name="to1" href="#1">1</a></sup>

与浪费我们的生命[艰难地做这些体力活](http://reliancehvg.co.in/store/images/P/hard%20way_dvd1.jpg)相比，使用 [PaintCode](http://www.paintcodeapp.com/) 会更轻松一点。这是一个超级简单易用的程序，能让我们通过简单的控制就能创造出这些图标，接着导出一堆我们可以用在应用里的 Objective-C 代码。

分分钟搞定。

当我尝试画一个形状的时候，Jake 用 PaintCode 描绘所有这些图标：

![](http://www.c4ios.com/images/cosmos/6/02.png)

所以，当他完成并导出这些图标后，我得到了一个包含十二个星座图标绘制方法的文件，就像这样：

```objective-c
//// 天秤座
UIBezierPath* bezier30Path = UIBezierPath.bezierPath;
[bezier30Path moveToPoint: CGPointMake(272.9, 181.68)];
[bezier30Path addLineToPoint: CGPointMake(265.4, 181.68)];
[bezier30Path addCurveToPoint: CGPointMake(254.1, 170.38) controlPoint1: CGPointMake(265.4, 175.48) controlPoint2: CGPointMake(260.3, 170.38)];
[bezier30Path addCurveToPoint: CGPointMake(242.8, 181.68) controlPoint1: CGPointMake(247.9, 170.38) controlPoint2: CGPointMake(242.8, 175.48)];
[bezier30Path addLineToPoint: CGPointMake(235.4, 181.68)];
```

这很棒。但是，因为我们在弄 C4，我要花时间将这些代码转化成我们的风格：

```swift
bezier.moveToPoint(Point(37.5, 11.3))
bezier.addLineToPoint(Point(30, 11.3))
bezier.addCurveToPoint(Point(18.7, 0), control1:Point(30, 5.1), control2:Point(24.9, 0))
bezier.addCurveToPoint(Point(7.4, 11.3), control1:Point(12.5, 0), control2:Point(7.4, 5.1))
bezier.addLineToPoint(Point(0, 11.3))

bezier.moveToPoint(Point(0, 20.2))
bezier.addLineToPoint(Point(37.5, 20.2))

```

更整洁。更具可读性。最终，我们现在能够使用 C4 创建，缩放我们的形状，以及在其上加载动画。<sup><a name="to2" href="#2">2</a></sup>

### 1.2. 星星的坐标

除了知道星座的形状，我们需要知道星星应该放在星座的什么位置。通过下图，观察星星之间的相对位置，Jake 开始了定义星星位置的过程：

![](http://www.c4ios.com/images/cosmos/6/03.png)

因为我们的设计是建立在这个图像上的，所以显然每个 `AstrologicalSign` 需要三组数据：小星星、大星星以及连线。

为了获得位置，Jake 用 [Sketch 3](http://bohemiancoding.com/sketch/) 来放置相对于 iPhone 4 大小的画布的小星星和大星星，等比例缩放以便它们的方向在 5 和 6 上看起来效果一样好。每一个位置坐标将会相对于屏幕中心，我们会基于十二星座位置集合中的一个根据它们确切的坐标来计算它们的偏移位置。

![显示 {0,0} 及其相对位置](http://www.c4ios.com/images/cosmos/6/04.png)

每个星座都有大的和小的点集数组...白羊座的数据集合（最简单的星座）是这样的:

```swift
let big = [
Point(58.0,-57.7),
Point(125.5,-17.7)]
sign.big = big

let small = [
Point(-134.5,-95.2),
Point(137.0,11.3)]
sign.small = small

```

*注意到为何有一些点是负数值？*例如，第一个**大**点是在屏幕中心的右上方。计算出每颗恒星的中心（或图像）的草图是很容易的，但是计算其基于中心的位置并没有那么简单...除非你使用 [Numbers](http://www.apple.com/ca/ios/numbers/)。我创建了一个表格，从 Sketch 图中导入数值，并将它们转换成基于图像大小渲染的视网膜坐标，从中我可以从画布的中心位置减去这些值。

例如:
1. Sketch 衡量 17x17 图像的左上角坐标为 {585.5, 582}
2. 基于 17x17 视网膜的中心坐标变成 {297.00, 295.3}
3. 然后与 iPhone 5 的画布中心位置 {160, 284} 相减（这是 Jake 在 Sketch 中设计的大小）
4. 最后中心点变成了 {125.5, -17.7}

星星之间的连线是任意的，所以我想出了一个依据星星位置的模式。Jake 按照这个模式通过给大星星和小星星编号来储存它们的位置，然后根据它们的顺序，写下它们的位置，如下：

```swift
let lines = [
[big[0],small[0]],
[big[1],big[0]],
[big[1],small[1]]]
sign.lines = lines
```

在 Jake 的最后 Numbers 文件中白羊座的位置长这样：

![](http://www.c4ios.com/images/cosmos/6/05.png)

### 1.3. 准备开始构建

既然我们知道我们要做什么，那么就可以开始构建提供星座的这个类了（Provider）。类很小，大约也就 40 行代码。每个星座的方法会占用许多空间，但方法里面没有逻辑，所以它们非常简单。

出发吧。


---

#### 脚注

<a name="1">1.</a> 当然伴随着一系列中断，停顿喝酒以及极其无聊的时刻。<a href="#to1">↩</a>

<a name="2">2.</a> 我本可以写一个脚本，但是我却花时间复制粘贴所有的点集，因为我可以这么做因为我只是想要「读」或者「知道」我将要操作的点集的信息。<a href="#to2">↩</a>