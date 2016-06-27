# Chapter 22 信息面板

和我们之前所做的比起来，这一章其实很简单。我们并不想在 app 中长篇大论地解释什么是 COSMOS，因此我们只是添加一个链接，让用户可以找到所有信息。

信息面板是一个包含一些文本和指向 C4 主页的超链接的弹出窗口，我们会按照设计图中的元素顺序来一个一个实现。

这是目标：

![](http://c4ios.com/images/cosmos/22/01.png)

在工程中打开 InfoPanel.swift 开始工作吧！

## 1. 画布

我们首先修改画布的背景颜色：

```swift
public override func setup() {
    canvas.backgroundColor = Color(red: 0, green: 0, blue: 0, alpha: 0.33)
}
```

## 2. Logo

修改下面的方法：

```swift
func createLogo() {
    // 创建一个图片，并在添加到画布前时放在合适的位置上
    let logo = Image("logo")!
    logo.center = Point(canvas.center.x,canvas.height/6.0)
    canvas.add(logo)
}
```

然后在 `setup()` 的末尾调用这个方法。

修改工程的 `WorkSpace` 为如下这样：

```swift
class WorkSpace: CanvasController {
    override func setup() {
        canvas.add(InfoPanel().canvas)
    }
}
```

看看效果：

![](http://c4ios.com/images/cosmos/22/02.png)


非常简单。一个简单的黑色蒙层把面板背后的东西都变暗，然后 logo 显示在屏幕的中间。

## 3. 文本

现在，创建一个方法来设置文本：

```swift
func createLabel() {
    let message = "COSMOS is a lovingly\nbuilt app created\nby the C4 team.\n\n\nWe hope you enjoy\ncruising the COSMOS.\n\n\nYou can learn how\nto build this app\n on our site at:"

    let text = UILabel()
    text.font = UIFont(name: "Menlo-Regular", size: 18)
    text.numberOfLines = 40
    text.text = message
    text.textColor = .whiteColor()
    text.textAlignment = .Center
    text.sizeToFit()
    text.center = CGPoint(canvas.center)

    canvas.add(text)
}
```

这个方法创建了 `UILabel`, 因为我们需要居中显示多行的文本，直接把它放在屏幕中央就可以了。

在 `setup()` 方法的末尾调用这个方法，运行一下：

![](http://c4ios.com/images/cosmos/22/03.png)

## 4. 超链接

下一步，要创建一个指向 C4 主页的超链接，我们需要创建一个方法来生成一个 `text shape`。首先需要添加如下的变量：

```swift
var link : TextShape?
```

然后，添加如下的方法：

```swift
func createLink() {
    let text = TextShape(text:"http://www.c4ios.com/docs", font: Font(name: "Menlo-Regular", size: 24))
    text.fillColor = white
    text.center = Point(canvas.center.x,canvas.height * 5.0/6.0)

    let a = Point(text.origin.x,text.frame.max.y+8)
    let b = Point(a.x + text.width + 1, a.y)

    let line = Line((a,b))
    line.lineWidth = 2.0
    line.strokeColor = C4Pink

    link = text
    canvas.add(link)
    canvas.add(line)
}
```

然后还是一样，在 `setup()` 方法中调用它。

这个方法创建一个 `text shape` 并将一个紫色的线到它的文字的底部。它的 Frame (Rect 类型) 有一个非常方便的属性 `max`, 可以返回这个 Frame 的右下角的位置。这就让我们添加链接的时候非常方便的就能得到它底部的位置。


> 我把线条的宽度 +1， 这样看起来更好看一些。

运行一下：

![](http://c4ios.com/images/cosmos/22/04.png)

## 5. 手势

现在我们已经把一切都布局好了，是时候给面板加一点交互了。我们需要做两件事：

1. 链接到 C4 主页
2. 关闭面板

### 5.1 关联手势

我们希望文本可以被点击，所以我们需要添加 `longPress` 手势，这样我们就能够捕捉到用户长按的手势。

下面是手势的基本设置：

```swift
func linkGesture() {
    let press = link?.addLongPressGestureRecognizer { locations, center, state in
        switch state {
        case .Began:
            self.link?.fillColor = C4Pink
        default:
            self.link?.fillColor = white
        }
    }
}
```

把这个方法添加到你的类里测试一下。你会发现当按住 `text shape` 不放时，它会变成粉红色，当放开后又会变回白色。我们要的就这效果，除了现在变粉色的时候会有点延迟。

要消除延迟，在 `let press = link?.addLongPressGestureRecognizer{ ... }:` 之后添加如下代码

```swift
press?.minimumPressDuration = 0.0
```

现在我们按住链接就会马上变化，很不错，不过我们还要加入更多功能。首先让我们看一下手势是如何工作的。手势一共有五种状态：

1. .Began 
2. .Changed
3. .Ended
4. .Cancelled
5. .Failed


对于我们这个手势，需要跟踪什么时候开始和结束（或者失败、取消），然后对应执行下述逻辑：

1. 当它开始时，变粉红色
2. 当它变化时，保持粉红色
3. 当他结束时，检查现在用户触摸的位置是否还在我们 `text shape` 的上方，如果是则打开 C4 的主页。否则就简单的把高亮取消


要处理这个逻辑，修改手势部分的代码为下面这样：

```swift
func linkGesture() {
    let press = link?.addLongPressGestureRecognizer { locations, center, state in
        switch state {
        case .Began, .Changed:
            self.link?.fillColor = C4Pink
        case .Ended:
            if let l = self.link where l.hitTest(center) { 
                UIApplication.sharedApplication().openURL(NSURL(string:"http://http://www.c4ios.com/cosmos/")!)
            }
            fallthrough
        default:
            self.link?.fillColor = white
        }
    }
    press?.minimumPressDuration = 0.0
}
```

需要注意的地方：

1. `.Begin .Changed` 分支处理了上面列表的第一点和第二点。
2. 语句 ` if let l = self.link where l.hitTest(center)` 把链接对象解包来查看手势是否结束，以及当前用户触摸位置是否仍在文本区域上方
3. 如果在，则通过调用 `.openURL()` 来打开 C4 的主页
4. `fallthrough` 方法允许我们将 `.Ended` 分支包含最终的默认处理
5. 在 `default` 分支中将链接变回白色。所以当手势结束、失败或者被取消时，都要执行这一步。

现在，在 `setup()` 中调用 `linkGesture()`，然后我们就能开始最后一节了。

### 5.2 隐藏手势

我们希望当我们点击链接之外的任何地方都能关闭面板。为了实现这个效果，我们创建一个动画和一个手势：

```swift
func hideGesture() {
    canvas.addTapGestureRecognizer { locations, center, state in
        self.hide()
    }
}
```

```swift
func hide() {
    ViewAnimation(duration: 0.25) { () -> Void in
        self.canvas.opacity = 0.0
    }.animate()
}
```

很直观，在 `setup()` 中调用 `hideGesture()`  就可以了。

运行并测试一下两个手势：

![](https://zippy.gfycat.com/EasygoingAggressiveBergerpicard.mp4)

### 5.3 显示

我们还需要一个方法来显示面板，下一章会用到。

添加如下代码到类里：

```swift
func show() {
    ViewAnimation(duration: 0.25) { () -> Void in
        self.canvas.opacity = 1.0
    }.animate()
}
```

## 6. 下一步

我们会创建 COSMOS 的最后一个主要组件。在下一章中我们会把各个组件组合成我们最后的 app。

这是 [InfoPanel.swift](https://gist.github.com/C4Framework/d14b7598cabc6ae95f8c) 的代码。

干得不错！