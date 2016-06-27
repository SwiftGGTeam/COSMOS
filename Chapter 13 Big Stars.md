# Chapter 13 大星星

这是最后一次和 `InfiniteScrollView` 子类打交道了，但是这次需要花一些功夫来完成最上图层背景的美化工作。

## 1. 添加星星

因为我们之前已经完成这个过程了，所以直接把代码复制到 `StarsBig.swift` 文件即可：

```swift
public override init(frame: CGRect) {
    super.init(frame: frame)
   
    var signOrder = AstrologicalSignProvider.sharedInstance.order
    contentSize = CGSizeMake(frame.size.width * (1.0 + CGFloat(signOrder.count) * gapBetweenSigns), 1.0)
    signOrder.append(signOrder[0])
    
    for i in 0..<signOrder.count {
        let dx = Double(i) * Double(frame.size.width  * gapBetweenSigns)
        let t = Transform.makeTranslation(Vector(x: Double(center.x) + dx, y: Double(center.y), z: 0))
        if let sign = AstrologicalSignProvider.sharedInstance.get(signOrder[i]) {
            for point in sign.big {
                let img = Image("7bigStar")!
                var p = point
                p.transform(t)
                img.center = p
                add(img)
            }
        }
    }
}
```

如此简单，哈哈。

## 2. 添加横线和标记

我们想要添加的横线（dash）和标记（marker）大致是这样的：

有一系列短的横线，接着，在每个星座标记的中心会有白色的标记。

对于聪明的我们来说，肯定不会创建一个蓝色的横线然后到处复制粘贴。

为了在屏幕底部高效地创建很多横线效果，我们可以创建一条有特定横线模式的线条。

> 你可以按照下面的步骤实现代码，或者在最后拷贝整个方法。

创建一个方法来添加横线和标记：

```swift
func addDashes() {
}
```

首先，我们创建一系列的点（point），它们可以定义两种线条：一种线条由短横线组成，一种线条由高横线组成。定义的点如下：

```swift
let points = (Point(0,Double(frame.maxY)),Point(Double(contentSize.width),Double(frame.maxY)))
```

每个线条和给定 scrollview 的宽度是一样的。

我们还要创建 `Line` 以及它的样式，像这样：

```swift
let dashes = Line(points)
dashes.lineDashPattern = [2,2]
dashes.lineWidth = 10
dashes.strokeColor = COSMOSblue
dashes.opacity = 0.33
dashes.lineCap = .Butt
add(dashes)
```

我们通过 `dashes.lineDashPattern` 设置横线的宽度为 `2pt`，同时设置横线间距为 `2pt`。这样我们就有了一个 `4pt` 的可重复使用的横线模式。

为了创建标记，编写如下方法：

```swift
func addMarkers() {
    for i in 0..<AstrologicalSignProvider.sharedInstance.order.count + 1 {
        let dx = Double(i) * Double(frame.width * gapBetweenSigns) + Double(frame.width / 2.0)

        let begin = Point(dx,Double(frame.height-20.0))
        let end = Point(dx,Double(frame.height))
        
        let marker = Line((begin,end))
        marker.lineWidth = 2
        marker.strokeColor = white
        marker.lineCap = .Butt
        marker.opacity = 0.33
        add(marker)
    }
}
```

在 `setup` 方法底部调用前面的两个方法。

> 循环中的 +1 可以确保我们生成第 13 个标记。

### 2.1. 符号名称

创建符号名称和小图标相当简单，我们将使用符号生成器（sign provider）和一些数学方法来计算文字形状。

照例，我会解释一下核心思想，然后你就可以复制代码了。

我们可以将这个问题分解成以下 4 个简单的步骤：

1. 创建小图标并返回 `Shape`
2. 创建小符号标题并返回 `TextShape`
3. 创建角度值（degree）并返回 `TextShape`
4. 排版所有内容

### 2.2. 小符号

为了创建小符号，添加如下的代码：

```swift
func createSmallSign(name: String) -> Shape? {
    var smallSign : Shape?
    // 从 provider 中获得一个符号，然后对其添加样式
    if let sign = AstrologicalSignProvider.sharedInstance.get(name)?.shape {
        sign.lineWidth = 2
        sign.strokeColor = white
        sign.fillColor = clear
        sign.opacity = 0.33
        // 缩放
        sign.transform = Transform.makeScale(0.66, 0.66, 0)
        smallSign = sign
    }
    return smallSign
}
```

我们创建了一个方法，输入的参数是星座符号的名字，类型为字符串。该方法尝试从 `signProvider` 中根据输入的名字获取对应的 `AstrologicalSign`。如果名字正确的话，以下代码会返回一个当前符号的图标：

```swift
if let sign = AstrologicalSignProvider.sharedInstance.get(name)?.shape {
}
```

接着，设置符号图标的样式，并改变图标规模为原来的 33%，最后返回。

### 2.3. 符号的标题

创建符号标题需要如下的代码：

```swift
func createSmallSignTitle(name: String, font: Font) -> TextShape {
    let text = TextShape(text:name, font:font)
    text.fillColor = white
    text.lineWidth = 0
    text.opacity = 0.33
    return text
}
```

这个方法会设置好标题及字体。

### 2.4. 符号的角度

以下步骤更简单了：

```swift
func createSmallSignDegree(degree: Int, font: Font) -> TextShape {
   return createSmallSignTitle("\(degree)°", font: font)
}
```

该方法根据传入的度值（`Int`类型），创建符号的标题，再调用之前创建标签的方法。

### 2.5. 摆放所有内容

用之前的三个方法创建好符号和标签之后，我们要使用一个方法遍历每个星座符号的名字和位置。

首先创建一个方法，使用滚动视图作为输入，创建一系列在每个迭代中会使用的默认值。

```swift
func addSignNames() {
    var signNames = AstrologicalSignProvider.sharedInstance.order
    signNames.append(signNames[0])
    
    let y = Double(frame.size.height - 86.0)
    let dx = Double(frame.size.width * gapBetweenSigns)
    let offset = Double(frame.size.width / 2.0)
    let font = Font(name:"Menlo-Regular", size: 13.0)!
    
    for i in 0..<signNames.count {
        // 对每个符号和标签设置位置
    }
}
```

该方法获取符号的名字并添加到列表中（与我们之前在小星星、大星星那部分的操作相同）。

接着，创建一个 `y` 变量来保存每个符号的中心位置，创建一个 `dx` 变量来保存基于特定符号位置的符号和标签的偏移量。再创建一个 `font` 变量，它会被重复使用。然后，遍历所有标签的名字。

用来设置每个符号和标签的代码如下：

```swift
let name = signNames[i]

var point = Point(offset + dx * Double(i),y)
if let sign = self.createSmallSign(name) {
    sign.center = point
    add(sign)
}

point.y += 26.0

let title = self.createSmallSignTitle(name, font: font)
title.center = point

point.y+=22.0

var value = i * 30
if value > 330 { value = 0 }
let degree = self.createSmallSignDegree(value, font: font)
degree.center = point

add(title)
add(degree)
```

该方法获取符号的名字并计算中心点。接着，尝试创建一个符号图标，如果成功的话会将中心点添加到滚动视图里。

接着，以 `26` 的间隔增加中心点的位置。还为当前符号创建标题标签，居中，并添加到滚动视图里。

接着，以 `22` 的间隔增加中心点的位置并为符号确定度值。如果当前符号是第 13 个的话，度值是 360，而实际上我们需要的是 0（以至于可以覆盖第一个标签）。再创建度标签，居中，并添加到滚动视图里。

所有需要添加的代码如下：

```swift
func addSignNames() {
    // 获取所有星座符号名称
    var signNames = AstrologicalSignProvider.sharedInstance.order
    // 在数组的末尾添加第一个元素
    signNames.append(signNames[0])
    
    // 确定 y 的坐标
    let y = Double(frame.size.height - 86.0)
    // 计算相对当前屏幕的偏移量
    let dx = Double(frame.size.width * gapBetweenSigns)
    // 定义画布的中心位置
    let offset = Double(frame.size.width / 2.0)
    // 新建一个字体
    let font = Font(name:"Menlo-Regular", size: 13.0)!
    
    // 对每个名称
    for i in 0..<signNames.count {
        // 获取名称
        let name = signNames[i]

        // 计算符号的中心位置
        var point = Point(offset + dx * Double(i),y)
        // 基于名称获取符号，然后添加到屏幕中
        if let sign = self.createSmallSign(name) {
            sign.center = point
            add(sign)
        }

        // 给 y 轴上的偏移量增加那么一点
        point.y += 26.0

        // 为当前的符号名称添加一个标签
        let title = self.createSmallSignTitle(name, font: font)
        title.center = point

        // 给 y 轴上的偏移量增加那么一点
        point.y+=22.0

        // 计算当前的角度
        var value = i * 30
        // 如果大于 330，那么就变成 0
        if value > 330 { value = 0 }
        // 为角度创建一个标签
        let degree = self.createSmallSignDegree(value, font: font)
        degree.center = point

        add(title)
        add(degree)
    }
}
```

> 你可把上面的代码复制粘贴到你的工程里。

现在，添加如下代码到 `init` 方法里：

```swift
public override init(frame: CGRect) {
    //…
    addDashes()
    addMarkers()
    addSignNames()
}
```

就是这样！

可以在[这里](https://gist.github.com/C4Framework/4d909b3f3fe6b8a94a47)下载 StarsBig.swift。

## 3. 测试之

用如下代码测试，你就可以看到下面的图：

```swift
canvas.backgroundColor = COSMOSbkgd
let bigStars = StarsBig(frame: view.frame)
canvas.add(bigStars)
```

![](http://c4ios.com/images/cosmos/13/01.png)
