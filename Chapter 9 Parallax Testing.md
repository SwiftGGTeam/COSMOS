# Chapter 9 调试视差图层（可选）

本章将向你简单介绍最初是如何测试背景（速度，层级等）的，且这一章是可选的。如果你想知道具体是怎么操作的，那么强烈推荐你一步步看下去。

## 1. 视差背景

首先，创建一个名叫 `ParallaxBackground` 的文件，并导入 `UIKit`（而不是导入 `Foundation`）。并在该文件中，添加一个和文件名相同的类（此类需继承自 CanvasController）。新的文件应该看起来是这样的：

```swift
class ParallaxBackground : CanvasController {
}
```

接下来，在 `WorkSpace` 中添加一个 `background` 变量，能用来初始化 `ParallaxBackground` 的实例：

在继续下去之前，我们先在 `WorkSpace` 外部建立一些颜色的常量以便整个工程都能使用。你的 `WorkSpace` 现在看起来应该是这样的：

```swift
import UIKit
import C4

let COSMOSprpl = Color(red:0.565, green: 0.075, blue: 0.996, alpha: 1.0)
let COSMOSblue = Color(red: 0.094, green: 0.271, blue: 1.0, alpha: 1.0)
let COSMOSbkgd = Color(red: 0.078, green: 0.118, blue: 0.306, alpha: 1.0)

class WorkSpace: CanvasController {
   var background = ParallaxBackground()
   
   override func setup() {
       canvas.add(background.canvas)
   }
}
```

> 和 WorkSpace 类似，背景对象也是 CanvasController 类的子类。因此，背景对象也有 canvas 属性（其实是一个 view），我们才能使用 background.canvas

完成以上这些之后，我们来看看添加具有视差的背景大致要几步：

1. 创建速度数组，以标明每个图层移动的快慢
2. 基于各自的速度创建并添加所有的图层
3. 为创建视差效果而添加一个上下文和观察者
4. 为一些标签测试滚动效果

以上这些操作都是在 WorkSpace 中完成的，接下来的操作都将在 ParallaxBackground 文件中实现。

## 2. 创建速度数组和滚动视图数组

这一步非常简单。为之后的操作创建一个 `CGFloat` 为元素类型的数组。

在类中添加如下的变量：

```swift
let speeds : [CGFloat] = [0.08,0.0,0.10,0.12,0.15,1.0,0.8,1.0]
```

我们从设计文件中获取到数值，并按顺序添加。最底层的速度为 `0.08`，是数组下标为 `[0]` 的第一个元素；而最上层的速度是最后一个元素。

我们将数组的类型设置为 `: [CGFloat]`，因为我们会将这些值直接赋值给继承自 `UIScrollView` 的视图。事先知道需要的类型是 `CGFloat`，而不用在使用 `Double` 的时候去转型了。

最后，添加一个元素为 `InfiniteScrollview` 类型的数组。之后的操作中会用到这个变量，我们先放在前面。

```swift
lazy var scrollviews = [InfiniteScrollView]()
```

你的类现在应该是这样子的：

```swift
class ParallaxBackground : CanvasController {
   let speeds : [CGFloat] = [0.08,0.0,0.10,0.12,0.15,1.0,0.8,1.0]
   var scrollviews = [InfiniteScrollView]()
}
```

## 3. 创建基于速度的图层

因为我们已经创建了 `InfiniteScrollView` 类，所以我们可以直接开始了。添加所有图层最简单的方法是一个个创建和添加，但是这样效率太低了，而且我们都已经知道各图层滚动速度是不同的。

![](http://www.c4ios.com/images/cosmos/9/01.png)

上面的这张图片展示了滚动视图是如何跟随速度改变而显示的。与最上层移动整个宽度相比，速度为 `0.08` 的第一层只需要移动最上层宽度的 8% 作为 `contentSize` 即可。这就意味着我们不需要为所有的图层都创建全尺寸的内容大小。

### 3.1. 动态图层的内容大小

我们想要根据各自的速度创建图层，而且，最好是有一个方法只要传入正确的参数就可以返回给我们。

这个方法就是：

```swift
func createLayer(speed: CGFloat) -> InfiniteScrollView {
   return InfiniteScrollView()
}
```

这可以作为 setup() 方法里创建动态图层的方法。创建的 setup() 方法如下：

```swift
override func setup() {
   for i in 0..<speeds.count {
  let layer = createLayer(speeds[i])
       canvas.add(layer)
  scrollviews.append(layer)
   }
}
```

我们在新的图层上添加画布（canvas）和滚动视图数组的原因主要有两个：一个是想在屏幕上显示，另一个是之后会用到。

运行之。

我们什么也没有看到，因为创建了没有内容的对象。因此，我们来修改前面提到的 `createLayer` 方法。

### 3.2. 修改 createLayer

首先，我们知道一共会有 12 个标志，因此创建一个 `let signCount`。

添加到你的类中：

```swift
let signCount : CGFloat = 12.0 
```

> 同样地，因为是赋值给 UIKit 对象的，所以我们指定的类型是 CGFloat。而且，将全局变量放置在类的顶部。

因为我们只是简单的测试我们写的 `createLayer` 方法，所以添加一些数字就可以了。

获取视图的框架大小，然后来初始化图层，如下：

```swift
func createLayer(speed: CGFloat) -> InfiniteScrollView {
   let frame = view.bounds
   let layer = InfiniteScrollView(frame: frame)
   return layer
}
```

没有直接返回 `InfiniteScrollView` 是为了在返回前修改图层内容的大小。

接下来，我们修改一下图层的内容大小，如下：

```swift
func createLayer(speed: CGFloat) -> InfiniteScrollView {
   let frame = view.bounds
   let layer = InfiniteScrollView(frame: frame)
   let contentSize = CGSizeMake(frame.size.width * 2.0 * signCount, frame.size.height)
   layer.contentSize = contentSize
   return layer
}
```

现在，为了在屏幕上显示，添加一些标签。在我们 `return` 之前，添加 `repeat` 来将这些标签充满整个 `scrollviews` 。

```swift
let dx = Double(layer.contentSize.width) / 100.0
var center = Point(dx, canvas.center.y)
repeat {
   let label = TextShape(text: "\(scrollviews.count)")
   label.center = center
   layer.add(label)
   center.x += dx
} while center.x < Double(layer.contentSize.width)
```

这一步创建了一个偏移变量 `dx`，会给我们基于图层 `contentSize` 的 20 个标签。接着，创建了一个 `center` 变量将用它的 x 位置标识每个添加的标签。最后，用 `repeat` 循环直到中心位置超出图层的内容。循环内，创建一个基于 `scrollviews` 数组当前元素数字的新标签（所以第一层的下标是 `0`）。

运行你的应用是这样的：

![](http://www.c4ios.com/images/cosmos/9/02.png)

看到所有的数字都叠在一起了吗？这是因为每层的 `contentSize` 都是相同的。是时候将 `speed` 变量派上用场了。

修改如下：

```swift
let contentSize = CGSizeMake(frame.size.width * 2.0 * signCount * speed, frame.size.height)
```

现在你整个 ParallaxBackground 文件看起来应该是这样的：

```swift
import UIKit

class ParallaxBackground : CanvasController {
    let speeds : [CGFloat] = [0.08,0.0,0.10,0.12,0.15,1.0,0.8,1.0]
    var scrollviews : [InfiniteScrollView]!
    let signCount : CGFloat = 12.0
    
    override func setup() {
        scrollviews = [InfiniteScrollView]()
        for i in 0..<speeds.count {
            let layer = createLayer(speeds[i])
            canvas.add(layer)
            scrollviews.append(layer)
        }
    }
    
    func createLayer(speed: CGFloat) -> InfiniteScrollView {
        let frame = view.bounds
        let layer = InfiniteScrollView(frame: frame)
        let contentSize = CGSizeMake(frame.size.width * 2.0 * signCount * speed, frame.size.height)
        layer.contentSize = contentSize

        let dx = Double(layer.contentSize.width) / 100.0
        var center = Point(dx, canvas.center.y)
        repeat {
            let label = TextShape(text: "\(scrollviews.count)")!
            label.center = center
            layer.add(label)
            center.x += dx
        } while center.x < Double(layer.contentSize.width)
        
        return layer
    }
}
```

你的应用应该是这样的：

![](http://www.c4ios.com/images/cosmos/9/03.png)

看起来不爽吧，让我们把它们根据 contentSize 和层数分开来吧：

```swift
let dx = Double(layer.contentSize.width) / 100.0
let dy = Double(layer.contentSize.height) / Double(speeds.count+1)
var center = Point(dx, Double(scrollviews.count + 1) * dy)
let fontSize = Double(scrollviews.count + 1) * 6.0
let font = Font(name:"AvenirNext-DemiBold", size: fontSize)!
repeat {
    let label = TextShape(text: "\(scrollviews.count)", font: font)!
    //...
}
```

现在看起来好多了：

![](http://www.c4ios.com/images/cosmos/9/04.png)

> 看到 1 很奇怪的靠在左边了吗？这是因为这一层的速度是 0.0...这一层不会移动，所以它的 contentSize 就是 0。

当你滚动的时候：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/UnnaturalObedientKillifish.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 4. 连接起来吧

之前做过的事情再让我们做一次！我们会为创建基于图层速度的视差效果而添加上下文和观察者。

首先，为类添加一个新属性：

```swift
var scrollviewOffsetContext = 0
```

这个变量会指出观察我们图层偏移量的上下文。

从敲出 `observeValueForKeyPath` 开始，当你敲了 `b` 时，Xcode 会提示 如下：

![](http://www.c4ios.com/images/cosmos/9/05.png)

敲 `return` 回车就可以了。

现在，添加如下代码：

```swift
if context == &scrollviewOffsetContext {
   print("top layer is scrolling")
}
```

在 `setup()` 方法里创建视图之后添加以下的代码来连接最上层的图层。

```swift
if let topLayer = scrollviews.last {
    topLayer.addObserver(self, forKeyPath: "contentOffset", options: .New, context: &scrollviewOffsetContext)
}
```

现在的类应该看上去是这样的：

```swift
class ParallaxBackground : CanvasController {
   var speeds ...
   var scrollviews ...
   let signCount ...
    
   var scrollviewOffsetContext = 0
    
   override func setup() {
       for i in 0..<speeds.count {
      ...
       }   
       if let topLayer = scrollviews.last {
           topLayer.addObserver(self, forKeyPath: "contentOffset", options: .New, context: &scrollviewOffsetContext)
       }
   }
    
   func createLayer(speed: CGFloat) -> InfiniteScrollView {
   }

   override func observeValueForKeyPath(keyPath: String?, ofObject object: AnyObject?, change: [String : AnyObject]?, context: UnsafeMutablePointer<Void>) {
       if context == &scrollviewOffsetContext {
           print("top layer is scrolling")
       }
   }
}
```

运行之。随着滚动，Xcode 控制台会跳出一些数字。

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/QualifiedSpitefulBluebird.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

现在，将观察的方法与其他的图层连接起来。

把

```swift
print("top layer is scrolling")
```

替换为

```swift
let sv = object as! InfiniteScrollView
let offset = sv.contentOffset

for i in 0..<scrollviews.count-1 {
   let layer = scrollviews[i]
   layer.contentOffset = CGPointMake(offset.x * speeds[i], 0.0)
}
```

代码中获取到观察的对象后，转型成 `InfiniteScrollview` （我们可以使用 `!` 来强制转型，因为我们确定知道它是什么类型的）。接着，获取到偏移量，并用其计算其余图层的偏移量。

最后，用循环遍历所有图层并基于速度修改 `contentOffset` 的值。

现在是这样子的：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/UltimateWellmadeAnkole.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

这里逻辑上有个错误。我们一开始说：如果速度是主图层的 80%，就只需要画布（canvas）是宽度的 80% 就够了。这是对的。

但是，当主图层滚动到画布边缘时。视图会停止滚动，因为只有内容在框架内时才能滚动。

![需要为每个画布增加 20% 的长度](http://www.c4ios.com/images/cosmos/9/06.png)

按理说，80% 主画布大小的画布只能滚动到 `80% - visibleFrameWidth`。因此，我们需要将框架的宽度添加到速度低于 1.0 的画布上。

如果要修复这个 bug，我们就要回到 `createLayer()` 方法，在创建 contentSize 和赋值之间添加一个判断条件：

```swift
let contentSize = ...
contentSize.width += speed < 1.0 ? view.frame.width : 0
layer.contentSize = contentSize
```

> 确保你把 `let contentSize = ...` 改成了 `var`

我们将使用一个技巧，让我们可以根据判断条件只用一行代码为变量赋值。

```swift
speed < 1.0 ?
```

意思就是：「这个 speed 是否小于 1.0？」

还有这个：

```swift
view.frame.width : 0
```

意思是如果判断为 `true` 就返回 `view.frame.width` 否则就是 `0`。

现在，都照我们预想那样了：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/EasyFemaleHarvestmen.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

[ParallaxBackground.swift](https://gist.github.com/C4Framework/f670e2fb15450514b776) 可以从这里下载。

## 5. 总结

这是我第一次测试以不同的速度产生视差效果。效果还不错：我们知道了每一层的内容是如何滚动的，以及所有图层针对最上层移动是如何响应的。

即使当前方法是有效的，我们也不会在应用使用，之后的章节中会讲到有什么区别。

现在，你可以从你的工程中删除整个类了，因为我们再也不会使用它了。

让我们继续吧！

