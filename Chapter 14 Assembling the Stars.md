# Chapter 14 组合这些星星

现在，是时候将星星的 8 层背景组合起来了。8 层分别是：

1. 黑背景中的星星
2. 小花饰
3. 背景中的小星星
4. 背景中的中星星
5. 背景中的大星星
6. 线条
7. 小星座的星星
8. 大星座的星星

我们接下来要完成的是：

1. 创建一个速度数组，可以让我们知道每一层的移动速度
2. 基于每一层的速度，创建并添加所有的层
3. 为创建视差效果而添加上下文和观察器
4. 添加 snapping
5. 添加线条的动画效果

## 1. 创建速度和滚动视图数组

这一步很简单。创建一个 `CGFloat` 的数组以备接下来的步骤使用。

打开 `Stars.swift` 并添加如下变量：

```swift
let speeds : [CGFloat] = [0.08,0.0,0.10,0.12,0.15,1.0,0.8,1.0]
```

我们把值从设计文件中取出来并按底层的速度顺序添加它们，0.08 是数组中的第一个元素（例如，[0]），最后一个元素是顶层的速度。

我们使用 `: [CGFloat]` 是因为数组里的元素可以直接赋值给 `UIScrollView` 的子类，而且之后要用的是 `CGFloat` 类型，因此不用转换成 `Double` 类型。

最后，添加一个元素类型为 `InfiniteScrollview` 的数组。之后会用到这个变量，所以先在前面声明。

```swift
var scrollviews : [InfiniteScrollView]!
```

> 如果你之前已经看过 `ParallaxBackground` 那章，你就会发现从这里开始代码有点不同了。

接下来，我们也需要线条的图层，小星星和大星星。因此，创建这些变量如下：

```swift
var signLines : SignLines!
var bigStars : StarsBig!
var snapTargets : [CGFloat]!
```

你的类大概是这个样子：

```swift
class Stars : CanvasController, UIScrollViewDelegate {
    let speeds : [CGFloat] = [0.08,0.0,0.10,0.12,0.15,1.0,0.8,1.0]
    var scrollviews : [InfiniteScrollView]!
    var signLines : SignLines!
    var bigStars : StarsBig!
    var snapTargets : [CGFloat]!

    override func setup() {
    }
}
```

## 2. 创建图层

我们已经创建了不同星星背景的类（都是 `InfiniteScrollview` 的子类），它们马上就能派上用场了。添加图层最简单的方式就是按照每个图层的速度和图片单独创建并添加。

![](http://c4ios.com/images/cosmos/14/01.png)

上一张图片展示了不同速度下滚动视图内容真正被看到的情况。与包含整个宽度的最上层相比，速度为 0.08 的第一层只需要最上层宽度的 8% 作为 `contentSize` 即可。

### 2.1. 小装饰

还有一个图层我们没有考虑：小装饰。它是一张位于其他图层后面，但在第一层上面的图片，它主要是为了展现层次感。因为该装饰层不会移动，所以不用单独为它创建类。

然而，因为我们其他的图层都是继承自 `InfiniteScrollview` 的，我们也对该花饰层这么干。添加如下的方法：

```swift
func createVignette() -> InfiniteScrollView {
    let sv = InfiniteScrollView(frame: view.frame)
    let img = Image("1vignette")!
    img.frame = canvas.frame
    sv.add(img)
    return sv
}
```

### 2.2. 前 5 个图层

以下是 `setup()` 中创建前 5 个图层的代码：

```swift
canvas.backgroundColor = COSMOSbkgd

scrollviews = [InfiniteScrollView]()
scrollviews.append(StarsBackground(frame: view.frame, imageName: "0Star", starCount: 20, speed: speeds[0]))
scrollviews.append(createVignette())
scrollviews.append(StarsBackground(frame: view.frame, imageName: "2Star", starCount: 20, speed: speeds[2]))
scrollviews.append(StarsBackground(frame: view.frame, imageName: "3Star", starCount: 20, speed: speeds[3]))
scrollviews.append(StarsBackground(frame: view.frame, imageName: "4Star", starCount: 20, speed: speeds[4]))
```

> 同时，也为背景添加了颜色。

### 2.3. 后 3 个图层

后 3 个图层与第 6 和第 8 层不同，不能放在变量中。主要是因为之后我们会在最上层（也就是有线条那层）做一些展示效果。

添加如下代码到 `setup()` 中：

```swift
signLines = SignLines(frame: view.frame)
scrollviews.append(signLines)

scrollviews.append(StarsSmall(frame: view.frame, speed: speeds[6]))

bigStars = StarsBig(frame: view.frame)
scrollviews.append(bigStars)

for sv in scrollviews {
    canvas.add(sv)
}
```

我们创建了 `signLines` 变量，接着添加到滚动视图数组中。创建 `bigStars` 变量后，接着添加小星星。

最后，我们遍历 scrollviews 变量，将其中的元素添加到 canvas 中。

## 3. 观察大星星

接下来的步骤将实现滚动时根据最上层来实时更新其他层的位置。

> 如果你想查看更多观察和上下文的内容，可以阅读 ParallaxBackground 章节。

添加如下变量到你的类中：

```swift
var scrollviewOffsetContext = 0
```

接着，在创建 `bigStars` 后把如下两行代码添加到 `setup()` 方法中：

```swift
bigStars.addObserver(self, forKeyPath: "contentOffset", options: .New, context: &scrollviewOffsetContext)
bigStars.delegate = self
```

就其本身而言，这并不会有什么效果。为了看到动作的流程，我们要添加一个 `observeValueForKeyPath` 方法来确定图层如何滚动。

```swift
override func observeValueForKeyPath(keyPath: String?, ofObject object: AnyObject?, change: [String : AnyObject]?, context: UnsafeMutablePointer<Void>) {
    if context == &scrollviewOffsetContext {
        let sv = object as! InfiniteScrollView
        let offset = sv.contentOffset
        for i in 0..<scrollviews.count-1 {
            let layer = scrollviews[i]
            layer.contentOffset = CGPointMake(offset.x * speeds[i], 0.0)
        }
    }
}
```

将你的工程的 WorkSpace 切换到如下的代码：

```swift
import UIKit

// 这三个颜色会在 app 中使用到，所以把他们放到了项目层的变量中
let COSMOSprpl = Color(red:0.565, green: 0.075, blue: 0.996, alpha: 1.0)
let COSMOSblue = Color(red: 0.094, green: 0.271, blue: 1.0, alpha: 1.0)
let COSMOSbkgd = Color(red: 0.078, green: 0.118, blue: 0.306, alpha: 1.0)


class WorkSpace: CanvasController {
    var background = Stars()
    var stars = Stars()
    
    override func setup() {
        
        canvas.backgroundColor = COSMOSbkgd
        canvas.add(stars.canvas
        
    }
}
```

看起来还不错的样子：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/DefinitiveTeemingHaddock.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

还有两件事情需要去做：

1. 让星座定位到特定位置
2. 当定位时，线条有进入或移出的动画效果

## 4. 定位

Jake 的设计考虑了以下的行为：

> 当一个符号在屏幕上且用户让滚动停止时，符号应该要定位到屏幕中心。

为了实现这个功能，我们首先要回答以下两个问题：

1. 我们怎么知道视图是否已经定位到特定位置
2. 我们怎么知道滚动什么时候停止

第二个问题依赖于第一个问题，因此我们先来解决第一个问题。

首先，创建一个保存目标点的数组：

```swift
var snapTargets : [CGFloat]!
```

接着，创建如下方法：

```swift
func createSnapTargets() {
    snapTargets = [CGFloat]()
    for i in 0...12 {
        snapTargets.append(gapBetweenSigns * CGFloat(i) * view.frame.width)
    }
}
```

该方法为每个星座符号添加中心的 `x` 轴位置。

该方法会在 `setup()` 方法中被调用：

```swift
override func setup() {
   // 一堆其他东西...
   createSnapTargets()
} 
```

接着，创建一个方法，以偏移量为输入，并判断视图是否需要定位。

```swift
func snapIfNeeded(x: CGFloat, _ scrollView: UIScrollView) {
    for target in snapTargets {
        let dist = abs(CGFloat(target) - x)
        if dist <= CGFloat(canvas.width/2.0) {
            scrollView.setContentOffset(CGPointMake(target,0), animated: true)
            return
        }
    }
}
```

该方法遍历 `snapTargets` 里的所有目标。并为每个目标计算与当前 `x` 轴位置的距离，如果距离小于屏幕宽度的一半的话，就需要定位，然后设置当前滚动视图的内容偏移量给对应的目标。

> 该方法有一个 `return` 语句，主要的目的就是在恰当的时候可以跳出循环，而不继续执行下去（比如，当前目标在列表中是第一个，那就没必要执行剩下的 12 个了）。

下面需要判断何时调用 `snapIfNeeded` 方法。

有两种情况：

1. 用户拖动地很快，滚动视图自己持续在那边滚动，然后减速，最后停止滚动。
2. 用户拖动地很慢，且慢慢将手指提起离开屏幕，此时滚动视图不需要减速。

`UIScrollview` 类里自带的一些方法可以帮主我们。分别是 `scrollViewDidEndDecelerating` 来解决第一种情况，`scrollViewDidEndDragging` 来解决第二种情况，两者都是代理中的方法。

添加第一个代理方法，并修改代码如下：

```swift
func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
    snapIfNeeded(scrollView.contentOffset.x, scrollView)
}
```

该方法会在滚动视图停止滚动时被隐式调用。当它被调用时，方法会获取当前视图的偏移量，并作为输入参数传递给 `snapIfNeeded` 方法。

同时，添加第二个代理方法：

```swift
func scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool) {
    if decelerate == false {
        snapIfNeeded(scrollView.contentOffset.x, scrollView)
    }
}
```

该方法会在滚动视图被用户停止拖动时隐式调用。如果用户的手指或拇指移动的足够慢，该方法会将 `decelerate` 参数设置为 `false`。这种情况下，我们需要立即检查视图是否需要定位。如果 `decelerate` 参数为 `true` 的话（用户在一个滑动手势后停止拖动滚动视图），上一个方法就会被调用了，因此我们什么也不用干。

为了使这两个代理方法执行，我们需要设置好最上层图层的代理。

首先，让整个 `ParallaxBackground` 类服从 `UIScrollViewDelegate` 代理：

```swift
class ParallaxBackground : CanvasController, UIScrollViewDelegate { ... }
```

其次，在 createBigStars 方法的返回（return）语句前设置代理。

```swift
func createBigStars() {
     //…
   addDashesMarker(bigStars)
   addSignNames(bigStars)
   bigStars.delegate = self
   return bigStars 
}
```

检查一下：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/BackBlaringEastrussiancoursinghounds.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 5. 线条的动画效果

最后一个美化的地方是当符号位于屏幕中心时显示线条的动画效果。我们需要指出什么情况下会触发（比如定位时），再让线条做出对应的动画效果。

有两种情况：

1. 线条应该在图形停止移动时出现
2. 线条应该在用户开始拖动时消失

第一个情况很简单。在 `snapIfNeeded()` 方法中，在 `return` 语句前添加如下代码：

```swift
delay(0.25, closure: { () -> () in
    self.signLines.revealCurrentSignLines()
})
```

该闭包会在动画开始前延迟 1/4 秒。

接着，我们会创建一个当用户开始拖动时触发的方法。这也很简单。添加如下方法到你的类中：

```swift
func scrollViewWillBeginDragging(scrollView: UIScrollView) {
    self.signLines.hideCurrentSignLines()
}
```

当用户开始拖动时， `scrollViewWillBeginDragging` 代理方法会被自动调用。因此，我们要做的就是隐藏这个线条。

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/IlliterateBonyAxisdeer.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 6. 下一步

最后一个步骤是将所有的东西都揉合到一起去。添加如下方法到你的 Stars 类中：

```swift
func goto(selection: Int) {
    let target = canvas.width * Double(gapBetweenSigns) * Double(selection)

    let anim = ViewAnimation(duration: 3.0) { () -> Void in
        self.bigStars.contentOffset = CGPoint(x: CGFloat(target),y: 0)
    }
    anim.curve = .EaseOut
    anim.addCompletionObserver { () -> Void in
        self.signLines.revealCurrentSignLines()
    }
    anim.animate()
    
    signLines.currentIndex = selection
}
```

我们会在后面的章节详细介绍这个方法，目前来说它只是接收一个下标，并把顶层图层移动到合适的位置。

## 7. 终章

星星背景的类就到此为止了。

可以下载 [Stars.swift](https://gist.github.com/C4Framework/8c30681420d7bea327b6) 的整个代码。