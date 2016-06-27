# Chapter 10 星星背景

我们接下来主要会在 `StarsBackground.swift` 文件里操作，当然也会在 `Stars.swift` 文件里测试我们的类。

我们有 4 个图层的背景是有星星，每一层的滚动视图都是可以无限滚动的。有 4 件事情要在开工前知晓：

1. 视图的框架应该有多大
2. 图层中星星图片的文件名
3. 添加到图层的星星数量
4. 将移动的图层的速度

我们可以构建一个图层，并拥有这 4 个元素。一旦我们建好并添加到应用中，我们就不再需要访问了。因此，我们可以一劳永逸的建一个初始化器。

打开 `StarsBackground.swift`，你会看到以下代码：

```swift
convenience public init(frame: CGRect, imageName: String, starCount: Int, speed: CGFloat) {
    self.init(frame: frame)
}
```

这个方法需要 4 个变量来构造图层。因此，我们只要给定参数就可以使用它了。

## 1. 符号之间的空隙

有一个小的设计规范需要我们在开始做之前考虑。星座之间会有很大一片空隙，每次用户都需要滚动一段才能从一个星座到达另一个星座。<sup><a name="to1" href="#1">1</a></sup>因此，我们需要一个可以计算内容大小的变量。

打开 `Stars.swift` 文件，添加如下代码至类的外部：

```swift
var gapBetweenSigns : CGFloat = 10.0
```

## 2. 修正内容大小

第一步是计算图层的内容大小。这一步很关键，因为我们要优化添加的资源。而且，因为这一层将以较慢的速度移动，所以也应该比其他层有较小的 contentSize。

![](http://www.c4ios.com/images/cosmos/10/01.png)

为此，我们需要三样东西：

1. 一个基于图层移动速度且修正过的框架大小
2. 一个计算好的大小，它包含了当前框架和下一个出现的标志之间所有的空间。我们命名为变量 `singleContentSize` 
3. 应用中标志的数量

![](http://www.c4ios.com/images/cosmos/10/02.png)

### 2.1 修正的 frame 大小

这个很简单了。添加以下变量到你的初始化方法中：

```swift
let adjustedFrameSize = frame.width * speed
```

如果速度比 1 小，我们将把 contentSize 调整到同样的大小。

### 2.2. 单个内容大小

图层 section 之间的空间是由框架大小和空隙数量（例如 frame 的宽度）决定的。计算起来也很简单直白：

```swift
let singleSignContentSize = adjustedFrameSize * gapBetweenSigns
```

### 2.3. 符号的数量

为了获取星座符号的数量，我们会访问之前写过的星座符号的 provider。如下：

```swift
let count = CGFloat(AstrologicalSignProvider.sharedInstance.order.count)
```

### 2.4. 计算内容大小

根据之前的三个变量，我们可以为图层计算真正的内容大小。如下：

```swift
contentSize = CGSizeMake(singleSignContentSize * count + frame.width, 1.0)
```

我们为内容大小添加了 `frame.width` 是因为我们需要图层在移动时超过末端。我们主要是在应用框架的空间内操作。

![](http://www.c4ios.com/images/cosmos/10/03.png)

### 2.5. 添加星星

我们将一次性遍历整个图层，随机地放置星星到每个框架内。并且，我们复制一份第一层的所有星星，然后添加到所有内容的后面，这样的话我们就可以让 `InfiniteScrollview` 滚动起来像是有无穷个星星了。

我们通过固定次数的循环来添加星星，为每个框架运行事先定好的次数。

设置 `contentSize` 后，添加如下循环：

```swift
for currentFrame in 0..<Int(count) {
   // 循环执行 12 次
}
```

接着，再添加一层循环进去，如下：

```swift
for currentFrame in 0..<Int(count) {
   for _ in 0..<starCount {
       // 添加一个星星
   }
}
```

如果我们想让每个框架都有 20 颗星星，我们每层就需要 12 * 20 = 240 颗星星。

我们需要计算当前框架的偏移量，如下：

```swift
for currentFrame in 0..<Int(signCount) {
   let dx = Double(singleSignContentSize) * Double(currentFrame)
   for _ in 0..<starCount {
       // 添加一个星星
   }
}
```

我们在代码中进行了 Double、Int 和 CGFloat 的类型转换。不幸的是，因为 Swift 是强类型的语言，所以不能让一个 Int 类型的值去加一个 Double 类型的值。因此，我们各自使用 CGFloat 和 Double 来对应 UIKit 和 C4 对象的数据类型。

以上，我们已经知道了每个框架中哪个位置需要偏移我们的图片。现在，我们需要计算在框架内的随机位置点了。

添加如下代码到内循环中：

```swift
let x = dx + random01() * Double(singleSignContentSize)
let y = random01() * Double(frame.size.height)
var pt = Point(x, y)
```

接着，在这个位置添加一个图片：

```swift
let img = Image(imageName)!
img.center = pt
add(img)
```

最终，如果 `currentFrame` 是第一个 frame 的话，就需要将其复制一份添加到第十三个框架，以示「无尽」之意（即最后多出来层叠的那一个上）。

在添加图片后，插入如下代码：

```swift
if currentFrame == 0 {
   pt.x += Double(signCount * singleSignContentSize)
   let img = Image(imageName)!
   img.center = pt
   add(img)
}
```

好了，我们已经可以用当前这个类来创建应用中的背景层了。

可以在[这里](https://gist.github.com/C4Framework/8e6c301aa84756952457)下载这个 StarsBackground.swift 类的最终形式代码。

### 2.6. 测试一下

为了测试这个类，将以下代码添加到你的工程中：

```swift
override func setup() {
    canvas.backgroundColor = COSMOSbkgd
    canvas.add(StarsBackground(frame: view.frame, imageName: "6smallStar", starCount: 20, speed: 1.0))
}
```

然后，你会发现如果向右拖的话，scrollview 看上去是无尽的。

完美！

---

#### 脚注

<a name="1">1.</a> 有一系列的原因...首先，如果让所有的星星都靠在一起会显得诡异，因为我们还要对星星做一些放大缩小的动画和让它们层叠在一起。其次如果间隔太小的话菜单的选择和动画就没必要实现了。再者，这些不同层级的动画只有当间隔大的时候才好看。最后一件，我们想 V2 版本的时候加一些交互元素，能够让大家在 app 中探索更多的东西。<a href="#to1">↩</a>