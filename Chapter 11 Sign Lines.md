# Chapter 11 星座符号间的连线

接下来的这个类是用来添加连接星星的紫色标记线的。

打开 `SignLines.swift` 后，你会发现它仍然是 `InfiniteScrollview` 的子类，并且和之前我们做过的类相比多了一些属性。

这一章的主要内容有：

1. 我们需要将图层中的所有线条组成一个数组。
2. 我们需要知道当前的标记（如，下标）。
3. 我们需要访问当前标记的所有线条。

## 1. 所有星星间的连线

大大小小的星星作为星空中的标记连接在一起。为了画出所有点之间的连线，我们从 `AstrologicalSign` 结构里把点取出来，并创建一个 `Line` 对象的数组来保存点之间的连线。

在开始添加连线之前，我们需要创建一个保存所有连线集合的数组（没错，我们要创建一个数组的数组）。

添加如下的属性到你的类中：

```swift
var lines : [[Line]]!
```

> 我们需要这个属性的原因主要是因为我们会将这些连线以动画的形式画出来。

## 2. 当前的连线

接下来，我们需要获取当前屏幕上出现的连线集合。我们可以创建一个下标和重载的属性来实现。实现如下：

```swift
var currentIndex : Int = 0
var currentLines : [Line] {
    get {
        let set = lines[currentIndex]
        return set
    }
}
```

这里有点小技巧。因为我们已经有所有连线了，就没必要创建多余的拷贝了，因此，只需要以当前连线的下标（`currentIndex`）从所有连线里取出来就可以了，用只读（`get`）的形式保存。

## 3. 初始化

正如之前创建的类一样，我们需要有初始化器来初始化对应的内容。因此，我们将会添加（逻辑上）一系列的 `init` 方法。

> 你不必担心 `init?(coder)` 方法，它放在那边只是为了让 Xcode 开心（傲娇脸）。

添加如下代码至 `super.init()` 后面：

```swift
let count = CGFloat(AstrologicalSignProvider.sharedInstance.order.count)
contentSize = CGSizeMake(frame.width * (count * gapBetweenSigns + 1), 1.0)

var signOrder = AstrologicalSignProvider.sharedInstance.order
signOrder.append(signOrder[0])
```

如上，我们获取到了图层的内容大小（content size）以及标记的顺序。接下来，我们将处理 `lines` 数组并加入到 canvas 中。

在 init 中加入如下的 `for` 循环：

```swift
lines = [[Line]]()
for i in 0..<signOrder.count {
    let dx = Double(i) * Double(frame.width * gapBetweenSigns)
    let t = Transform.makeTranslation(Vector(x: Double(center.x) + dx, y: Double(center.y), z: 0))
    if let sign = AstrologicalSignProvider.sharedInstance.get(signOrder[i]) {
        // 在这里对这些连线布局
    }
}
```

我们按顺序获取标记，并在循环中创建偏移（`dx`）和转变（`t`）。获取当前的 `AstrologicalSign` 并准备构建连线。

现在，在注释处添加如下的代码：

```swift
let connections = sign.lines
var currentLineSet = [Line]()
for points in connections {
    var begin = points[0]
    begin.transform(t)
    var end = points[1]
    end.transform(t)
    
    let line = Line((begin,end))
    line.lineWidth = 1.0
    line.strokeColor = COSMOSprpl
    line.opacity = 0.4
    line.strokeEnd = 0.0
    
    add(line)
    currentLineSet.append(line)
}
lines.append(currentLineSet)
```

首先，获取当前标记的所有连接（connections）。接着，创建一个空的数组来保存当前标记的所有的线条（lines），最后，遍历所有连接，构建每一条线条。

为了构建每条线条，需要获取当前连接的起点（`begin`）和终点（`end`），并加入之前计算好的转变量（`t`）。接着，设置好线条的样式，并将其添加到 scrollview 和当前线条的集合中。最终，添加 `currentLineSet` 到整个标记线条的列表中。

## 4. 动画

我们应用的设计主要是：当前线条的集合会在快速滑动进入屏幕时会有进入的动画效果，而在用户滚动出去后有消失出去的动画效果。为了实现这样的功能，我们需要添加两个动画的方法。

添加如下代码到类中：

```swift
func revealCurrentSignLines() {
    ViewAnimation(duration: 0.25) {
        for line in self.currentLines {
            line.strokeEnd = 1.0
        }
    }.animate()
}

func hideCurrentSignLines() {
    ViewAnimation(duration: 0.25) {
        for line in self.currentLines {
            line.strokeEnd = 0.0
        }
    }.animate()
}
```

这就结束了！

可以在[这里](https://gist.github.com/C4Framework/7b3497da5c4094e65f97)获取 `SignLines.swift` 完整的代码。

## 5. 测试之

为了测试我们线条的布局，在工程的 `WorkSpace` 中运行如下代码：

```swift
let linesLayer = SignLines(frame: view.frame)
canvas.add(linesLayer)
```

并确保你已经将 `SignLines` 中如下代码的注释去掉了：

```swift
//line.strokeEnd = 0.0
```

如果你滚动视图的话，你会看到所有不同的标记图形。以下是其中一些例子：

![](http://c4ios.com/images/cosmos/11/01.png)

> 记得将这些测试的代码删除掉，再进行下面的步骤。
