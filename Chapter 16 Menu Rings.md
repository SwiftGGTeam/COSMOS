# Chapter 16 圆环菜单

本章的主要目标是完成菜单的两种状态，我们会创建一些结构体，完成两种状态间的转换。例如，我们不再给一条厚线创建另外一个动画，而是创建一个数组，保存我们的目标 frame，在动画过程中更新圆环 frame 的大小。我们会在之后的演示中创建所需的结构体。

## 1. 圆环

这些线是菜单的骨架部分，我们把这个组件分解成几个部分：创建所有的圆环，设置开始状态，设置结束状态，最后添加动画。

![](http://www.c4ios.com/images/cosmos/16/01.png)

先创建厚线、细线、竖线和分割线。

### 1.1. 粗圆环

先实现粗圆环，这里有两个：一个是小的，直径 28 pt，另外一个是大的，直径 450 pt。

打开 `MenuRings.swift` 创建一个可变数组，存储粗圆环的尺寸。

```swift
var thickRingFrames : [Rect]!
```

接着，给圆环创建一个变量：

```swift
var thickRing : Circle!
```

接着，写一个方法来创建粗圆环，设置 targets（主体）的内外部的形状，接着把它们的 frames 添加到数组里：

```swift
func createThickRing() {
    // 创建两个形状
    let inner = Circle(center: canvas.center, radius: 14)
    let outer = Circle(center: canvas.center, radius: 225)
    // 存储每个位置的 frame
    thickRingFrames = [inner.frame,outer.frame]
}
```

之后我们会使用这个 frame 数组来实现我们的动画。现在嘛，我们先实现圆环的开始状态。把下列代码添加到 `createThickRing` 函数里最下方：

```swift
inner.fillColor = clear
inner.lineWidth = 3
inner.strokeColor = COSMOSblue
inner.interactionEnabled = false
thickRing = inner

canvas.add(thickRing)
```

### 1.2. 检查一下

为了查看一下你刚刚添加的代码的实际效果，需要在 WorkSpace 里的 `setup()` 方法添加下面的代码：

```swift
canvas.add(MenuRings().canvas)
```

App 现在看起来应该是下图这个样子：

![](http://www.c4ios.com/images/cosmos/16/02.png)

如果要看一下圆环处于最外层时的状态，在 `MenuRings.swift` 文件里把下面代码：

```swift
thickRing = inner
```

换成这样：

```swift
thickRing = inner
thickRing.frame = outer.frame
```

就能看到下图这样的效果了：

![](http://www.c4ios.com/images/cosmos/16/03.png)

在继续下一步之前，撤销刚刚做出的修改。

### 1.3. 细圆环

创建细圆环的过程也是一样的，不同之处在于有多个细圆环。先创建两个数组：

```swift
var thinRings : [Circle]!
var thinRingFrames : [Rect]!
```

我们给细圆环创建数组，因为需要记录五个细圆环。

设计稿也详细说明了开始状态下的内层圆环的直径是 8 pt，外层圆环的直径是 56 pt、78 pt、98 pt 和 156 pt。

创建一个方法，设置和创建全部的圆环，包括内层和外层的：

```swift
func createThinRings() {
    thinRings = [Circle]()
    thinRings.append(Circle(center: canvas.center, radius: 8))
    thinRings.append(Circle(center: canvas.center, radius: 56))
    thinRings.append(Circle(center: canvas.center, radius: 78))
    thinRings.append(Circle(center: canvas.center, radius: 98))
    thinRings.append(Circle(center: canvas.center, radius: 102))
    thinRings.append(Circle(center: canvas.center, radius: 156))
}
```

我们创建了一堆形状并添加到数组里，之后可以引用它们。

接下来写一个循环，遍历所有的，设置每个形状的风格。把下列代码添加到 `createThinRings` 方法里：

```swift
thinRingFrames = [Rect]()
for i in 0..<self.thinRings.count {
    let ring = self.thinRings[i]
    ring.fillColor = clear
    ring.lineWidth = 1
    ring.strokeColor = COSMOSblue
    ring.interactionEnabled = false
    if i > 0 {
        ring.opacity = 0.0
    }
    self.thinRingFrames.append(ring.frame)
}

for ring in thinRings {
    canvas.add(ring)
}
```

循环看起来很简单：遍历所有的圆环，执行一系列设置样式的代码。圆环按照从小到大的顺序添加到数组里，所以数组里的第一个元素成了内层第一个圆环（例如当菜单处于默认状态时）。一开始我们只想看内层圆环，所以把 i > 0 的圆环设置透明度为 0。我们把每个圆环的 frame 尺寸添加到 `thinRingFrames` 里，最后把所有的圆环添加到 canvas 里。

更新 `setup()` 方法，如下：

```swift
public override func setup() {
    createThickRing()
    createThinRings()
}
```

### 1.4. 检查一下

运行程序，应用看起来如下图所示：

![](http://www.c4ios.com/images/cosmos/16/04.png)

为了看一下外层圆环，在 `createThinRings()` 方法里，把下列代码：

```swift
if i > 0 
```

改成这样：

```swift
if i == 0
```

现在，App 看起来应该是这样的：

![](http://www.c4ios.com/images/cosmos/16/05.png)

 > 撤销刚刚做的修改。

### 1.5. 破折线

和上面两节做的事情一样，我们要创建一个数组来存储边框为破折线的圆。实际上破折线是最终要填到圆环上，所以我们不需要存储这些破折线。

把下列变量添加到类中：

```swift
var dashedRings : [Circle]!
```

创建三个方法：

```swift
func createShortDashedRing(){
}
func createLongDashedRing(){
}
func createDashedRings() {
    dashedRings = [Circle]()
    createShortDashedRing()
    createLongDashedRing()
    
    for ring in self.dashedRings {
        ring.strokeColor = COSMOSblue
        ring.fillColor = clear
        ring.interactionEnabled = false
        ring.lineCap = .Butt
        self.canvas.add(ring)
    }
}
```

我们需要创建长短两种竖线，粗细程度不同，每种竖线都有自己的模式。

### 1.6. 短破折线线圆环

短破折线的设置如下：

```swift
func createShortDashedRing() {
    let shortDashedRing = Circle(center: canvas.center, radius: 82+2)
    let pattern = [1.465,1.465,1.465,1.465,1.465,1.465,1.465,1.465*3.0] as [NSNumber]
    shortDashedRing.lineDashPattern = pattern
    shortDashedRing.strokeEnd = 0.995
    
    let angle = degToRad(-1.5)
    let rotation = Transform.makeRotation(angle)
    shortDashedRing.transform = rotation
    
    shortDashedRing.lineWidth = 0.0
    dashedRings.append(shortDashedRing)
}
```

这里实际上进行了很多设置，有的设置还依赖于其他的设计因素，所以我在这里将上述代码尽可能地分解：

1. 圆圈的直径是 `82+2`，我本来可以写 `84` 的，不过 `+2` 实际上值得是 `lineWidth` 的一半。<sup><a name="to1" href="#1">1</a></sup>
2. `pattern` 是 `1.462,....`，使用这个数字实际上是有两个原因的：a）圆圈可以分成 36 个部分，每个部分 10 度，b）`1.465*3` 表示间隙（例如每个长竖线之间间隔两个空格，外加一个额外的空格，一个三个空格），c）`as [NSNumber]` 必须要有，因为底层属性需要（例如不是 `[Double]`）。
3. 因为模式会从第一条线开始，所以我们需要旋转一点整个图形，这样看起来好像起始位置是空格了，旋转度为 `-1.5` 度，转换成弧度，创建 transform，应用到形状上。
4. 剩下的内容都简单易懂无需解释了。

### 1.7. 长破折线圆环

`longDashedRing` 方法如下：

```swift
func createLongDashedRing() {
    let longDashedRing = Circle(center: canvas.center, radius: 82+2)
    longDashedRing.lineWidth = 0.0

    let pattern = [1.465,1.465*9.0] as [NSNumber]
    longDashedRing.lineDashPattern = pattern
    longDashedRing.strokeEnd = 0.995

    let angle = degToRad(0.5)
    let rotation = Transform.makeRotation(angle)
    longDashedRing.transform = rotation

    let mask = Circle(center: longDashedRing.bounds.center, radius: 82+4)
    mask.fillColor = clear
    mask.lineWidth = 8
    longDashedRing.layer?.mask = mask.layer

    dashedRings.append(longDashedRing)
}
```

其实和之前的方法很相似，有几个不同之处：

1. 模式是 `[1.465,1.465*9.0]`，表示每个间隙之后有一个竖线，间隙的宽度比竖线宽 9x。
2. 旋转 5 度，将长竖线居中，正好在短竖线间隙的中间。
3. 最后一条线有微小的差异，最后一条线稍微可见，所以把 `strokeEnd` 的值从 `1` 调整成 `0.995`，隐藏一下。
4. 接着，创建遮罩...

更新 `setup()` 如下：

```swift
public override func setup() {
    createThickRing()
    createThinRings()
    createDashedRings()
}
```

### 1.8. 检查一下

要看圆环什么样子，需要做一下操作：

把这行代码：

```swift
shortDashedRing.lineWidth = 0.0
```

改成：

```swift
shortDashedRing.lineWidth = 4.0
```

把这行代码：

```swift
longDashedRing.lineWidth = 0.0 
```

改成：

```swift
longDashedRing.lineWidth = 12.0
```

运行，效果如下图：

![](http://www.c4ios.com/images/cosmos/16/06.png)

> 记得把上面的两个变动再改回去。

## 2. 遮罩

遮罩组件算是个小技巧，在塑造竖线的形状方面，能减轻工作量。默认情况下，竖线是画在中心的外围的，如果你前面已经有了一条 12pt 水平的线，那么上面和下面的线是 6pt。

![](http://www.c4ios.com/images/cosmos/16/07.png)

设计图显示，两个圆圈的初始状态下的直径都是一样的。我们想让竖线看起来像是从基线向外生长...所以，我们给多余的部分盖上遮罩，就看不到多余的部分了。

> 这也是属性是 12pt 的原因，但是在屏幕上看起来只有 6pt...因为砍掉了 6pt
 
遮罩是这样工作的：透过遮罩有颜色的地方可以看到下面的对象。所以，我们创建一个 8pt 的实线，它的直径要足够大，这样遮罩的边缘会碰到圆的基线并一直延伸到长竖线的顶部。

如下所示：

![](http://www.c4ios.com/images/cosmos/16/08.png)

欧耶！当你将遮罩应用到某个对象上时，它会基于对象的*内部*坐标系定位，这就是为什么我们需要用 `longDashedRing.bounds.center` 来确定遮罩的中心点。

## 3. 分割线

下一步是创建分割线，将每个图标分隔开来。

首先，创建如下变量：

```swift
var menuDividingLines : [Line]!
```

接着，在类里增加如下方法：

```swift
func createMenuDividingLines() {
    menuDividingLines = [Line]()
    for i in 0...11 {
        let line = Line((Point(),Point(54,0)))
        line.anchorPoint = Point(-1.88888,0)
        line.center = canvas.center
        line.transform = Transform.makeRotation(M_PI / 6.0 * Double(i) , axis: Vector(x: 0, y: 0, z: -1))
        line.lineCap = .Butt
        line.strokeColor = COSMOSblue
        line.lineWidth = 1.0
        line.strokeEnd = 0.0
        canvas.add(line)
        menuDividingLines.append(line)
    }
}
```

设置直线的步骤相当简单，我们知道图标内部和外部边缘直接的间隙是 `54pt`，那么我们要画的这条分割线也是这么长，设置分割线的风格，改变 `anchorPoint` 。

## 4. 锚点

每个可见的对象都有一个锚点，默认位置是在对象视图的居中位置。围绕这个点实现各种变形。例如，如果我只是让一个对象旋转某个角度（正如我在长短竖线那里所做的），那么整个对象都会围绕自己的锚点旋转。<sup><a name="to2" href="#2">2</a></sup>

![](http://www.c4ios.com/images/cosmos/16/09.png)

我想要的效果是，线的角度均匀地分布在两个圆圈之间，依赖于 `anchorPoint` 属性。我们可以计算每条线的旋转位置 `a` 和 `b`，不过这样创建效果可不太优雅。

我们需要做的是把 `anchorPoint` 位置偏移，这样我们可以在线的外面围绕一点旋转。唉，还是看图片更容易理解，一图胜千言，效果如下图：

![](http://www.c4ios.com/images/cosmos/16/10.png)

关于锚点的另外一件事情就是，它们的测量和对象视图的空间有关。具体说来，一个视图的中心点是 `{0.5,0.5}`，所以，现在我们只需要找出我们需要把锚点放在哪里，这样，`54pt` 的线就会出现在正确的位置了。

已经知道内圆的半径是 `102`（例如，倒数第二个细圆环），我们也知道线的宽度是 `54`，所以我们需要做的就是转换相关的坐标：

> 102/54 = 1.888
 
由于我们想让点在视图外部距左的距离为 `0`，我们需要把值设置成负数，也就是下面这行代码：

```swift
line.anchorPoint = CGPointMake(-1.88888,0)
```

方法中剩下的部分都很简单，把锚点居中，位于 canvas 的中心部分，然后旋转分割线，12 条线都进行同样的操作后，把它们添加到 canvas 和数组里（之后会玩出更多花样的）。

![](http://www.c4ios.com/images/cosmos/16/11.png)

V5。分割线已完成。

哦对了，别忘了，如果我们没有调整分割线的锚点，布局看起来应如下图所示：

![](http://www.c4ios.com/images/cosmos/16/12.png)

`setup()` 看起来应该是这个样子的：

```swift
override func setup() {
   self.createThickRing()
   self.createThinRings()
   self.createDashedRings()
   self.createMenuDividingLines()
}
```

## 5. 检查一下

如果想看到分割线，在 createMenuDividingLines 里进行修改：

把下面的代码删除：

```swift
line.strokeEnd = 0.0
```

改成：

```swift
line.strokeEnd = 1.0
```

> 或者注释掉也行。

效果如下图：

![](http://www.c4ios.com/images/cosmos/16/13.png)

如果你想到原来的样子，更改所有之前的变量，看一下圆环外部和直线的状态，效果如下图：

![](http://www.c4ios.com/images/cosmos/16/14.png)

 > 撤销刚刚做的修改，把直线设置成在里面的状态。

V5！

这些线看起来不错。

让我们继续下一章吧。

---

#### 脚注

<a name="1">1.</a> 我在写的时候就在想，为什么要这样写？不过之后又看了一遍代码之后，我意识到，我有点喜欢这样了，能我记住中心点需要调整一下，虽然 Jake 设计稿里明确直径是 82pt。<a href="#to1">↩</a>

<a name="2">2.</a> 注意我正在对 z 轴应用一个旋转，不过也可以应用在 x 或 y 轴上。<a href="#to2">↩</a>
