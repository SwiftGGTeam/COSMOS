# Chapter 5 无限滚动视图：实现

你需要做下面几件事情：
 
1. 实现 UIScrollView 的子类（可选）
2. 重写 layoutSubviews() 方法
3. 获取 currentOffset
4. 需要时，让视图再次显示开始或者结尾部分

> 实现 `UIScrollview` 的子类是可选的，因为这里已经存在一个 `InfiniteScrollview.swift` 文件了，你可以在这里编写代码。如果你选择跳过这部分，那你可以直接跳到*增加一些内容*这段。然而，对于之前没有做过的同学，我已经把创建子类的步骤写下在下面了。

## 1. 创建 UIScrollview 的子类

在 Xcode 里创建一个新的文件，可以用这种方法：

> File > New > File...

或者敲击快捷键：

> CMD + N
 
选择： 

> iOS > Source > Swift File
 
命名文件：
 
> InfiniteScrollView

文件创建后，将下面这行代码删除：
 
> import Foundation
 
换成这行代码：
 
> import UIKit
 
接着，输入 `class` 单词，在你输入完整个单词之前，你会看到类似下图的弹出框：
 
![显示自动补全的内容，有 Subclass 选项](http://www.c4ios.com/images/cosmos/5/05.png)

选择 **Swift Subclass** 敲击回车。Xcode 会给你刚刚创建的类自动提供空白来让你输入内容。

选择 **name** 然后输入 **InfiniteScrollView**。

点击 **Tab** 键，光标出现在 super class variable 这里，输入 `UIScrollView`。

再次点击 **Tab** 键，光标出现在属性和方法这里。

输入 `layoutSubviews`，输入的时候会出现自动补全，点击 **回车** 即可使用自动补全功能。

接下来，复制下面的代码，粘贴到你刚刚创建的新方法里：

```swift
super.layoutSubviews()
```

*完成了*。你创建了一个子类叫做 'InfiniteScrollView'，这个类和 `UIScrollView` 的功能一模一样，不过，还没开发什么特别的功能，所以，让我们来开发无限滚动效果。

另外，你的方法应该看起来和下面这个方法一样：

```swift
override func layoutSubviews() {
   super.layoutSubviews() // 调用父类的方法
}
```

### 1.1. 放到屏幕上

是时候来测试你新创建的类了，所以，回到 `WorkSpace` 文件，然后输入下列代码：

```swift
class WorkSpace: CanvasController {
    let infiniteScrollView = InfiniteScrollView() // 创建一个新的 InfiniteScrollview 对象
    override func setup() {
        // 根据 canvas 的 frame 创建 CGRect
        // 然后赋值给 infiniteScrollView，让 infiniteScrollView 的大小和 canvas 一样大
        infiniteScrollView.frame = CGRect(canvas.frame)
        
        // 把 infiniteScrollView 添加到 canvas 里
        canvas.add(infiniteScrollView)
    }
}
```

创建了一个名为 `infiniteScrollView` 的 `InfiniteScrollView` 变量，将其 frame 的等同于当前 canvas 的大小，然后添加到 canvas 里。

> 你需要把 canvas 的 frame 值（`Rect`）转换成 `CGRect`，因为 `InfiniteScrollView` 是 `UIKit` 的子类，只能处理 `CGRect` 和 `CGPoint` 类型。
 
继续...

点击：

> Build > Run
 
或者
 
> CMD + R
 
或者

> 点击 Xcode 上方的 play 按钮
 
模拟器出现了，里面没什么东西...实际上这里是有一个 view 的，只是你看不到，背景是透明的，你也不能向下滑动，毕竟里面没有内容，自然无法滑动。

### 1.2. 增加一些内容

为了测试你刚刚创建的 `InfiniteScrollView`，你需要添加一些视觉指示器。

将下面的代码复制粘贴到 `WorkSpace` 里：

```swift
func addVisualIndicators() {
    // 指示器的最大数量
    let count = 20
    
    // 每个指示器之间的间距
    let gap = 150.0
    
    // 初始化指示器的偏移量，因为我们要把指示器放在居中的位置
    let dx = 40.0
    
    // 计算 view contentSize 的总宽度
    let width = Double(count) * gap + dx
    
    // 创建主要的指示器
    for x in 0...count {
        // 给新的指示器创建中心点
        let point = Point(Double(x) * gap + dx, canvas.center.y)
        // 创建 textshape
        let ts = TextShape(text: "\(x)")
        ts.center = point
        // 添加到视图上
        infiniteScrollView.add(ts)
    }
}
```

然后，调用 `setup()` 方法，如下：

```swift
override func setup() {
    infiniteScrollView.frame = CGRect(canvas.frame)
    canvas.add(infiniteScrollView)
    addVisualIndicators()
}
```

**运行**，现在，模拟器上出现了很多连续的数字。

### 1.3. 内容 + 滚动

在 `infiniteScrolView` 里，还有些东西无法滚动，因为 view 实际上还不知道里面有什么内容，你必须告诉 view 你更新了一些内容，通过更新 contentSize 变量即可实现。

在循环后面加上这行代码：

```swift
infiniteScrollView.contentSize = CGSizeMake(CGFloat(width), 0)
```

现在 view 知道它内容的最大尺寸，view 可以滚动了。使用 `0` 作为内容的高度，因为你不需要 view 垂直滚动。

如果你滚动到最末端，你会注意到最后一个数字有点奇怪...这是因为内容的大小只是一个估计值。

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/LiveRepulsiveCarp.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

为了修复这个问题，你可以从下方两个解决方法中选一个：

1. 记录最新的原始位置
2. 想办法知道最后一个对象的结束位置

之后更新这个代码。

把下列删除：

> nfiniteScrollView.contentSize = CGSizeMake(CGFloat(width), 0)
 
替换成：

> infiniteScrollView.contentSize = CGSizeMake(CGFloat(width+gap), 0)
 
从循环体中得到最后一个变量时，你计算出中心点，然后 x 方位增加一个间隙，这样你就能看到最后 20 这个数字了。

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/RawBouncyKouprey.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

接下来，你要开始跟踪 `contentOffset` 的值。

> 不要担心，view 还是可以回弹的，过会儿你就会来解决你过会就会看到了...

### 1.4. 内容的偏移量

`UIScrollView` 的工作方式基本上是这样的：scrollview 要显示的内容远远大于自身 frame 的大小，在任何时间，用户看到的 content view 实际上滚动视图里的范围。

当用户滑动界面时，你需要找到内容的位置。

回到 InfiniteScrollView.swift 文件，更新 `layoutSubviews()` ，如下：

```swift
override func layoutSubviews() {
   // 调用 UIScrollView 的布局方法
   super.layoutSubviews()
   print(contentOffset.x)
}
```

`UIScrollView` 有一个 `contentOffset` 属性，所以你刚刚做的就是告诉 app ，只要调用 `layoutSubviews()` 方法，就打印出变量的 x 坐标。

**运行**应用，当你在模拟器上滑动时，你会看到 Xcode 调试视图中出现了一大堆的数字。

如果你看不到 Xcode 的调试台，敲击快捷键 **CMD+SHIFT+C**

或者，点击 Xcode 右上角的图片，就像下图一样：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/BarrenHastyIrukandjijellyfish.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

现在能跟踪到 scrollview 的 `contentOffset` 的 `x` 坐标了。

> 你可能会注意到，当你朝向开头滑动时，contentOffset.x 的值变成了负数，之后你会用到这个特点的，给你带来不少好处。

### 1.5. 无限循环的规则

现在需要实现无限循环的规则了。

更新 `layoutSubviews()` 方法，如下：

```swift
public override func layoutSubviews() {
    super.layoutSubviews()

    // 获取当前内容的偏移量（左上角）
    var curr = contentOffset
    // 如果 x 的值小于零
    if curr.x < 0 {
        // 更新 x 的值，复制 ScrollView 的结束位置
        curr.x = contentSize.width - frame.width
        // 设置 view 的内容偏移量
        contentOffset = curr
    }
        // 如果 x 值大于 width - frame width 
        //（比如，当右上角的点超出了当前的 contentSize.width）
    else if curr.x >= contentSize.width - frame.width {
        // 更新 x 的值，赋值 ScrollView 的开始位置
        curr.x = 0
        // 设置 view 的内容偏移量
        contentOffset = curr
    }
}
```

不管什么时候到了内容的边界位置，从另外一个方向，view 应该迅速跳过，这样才能有持续滚动的效果。

**运行**，自己看一下效果吧。

### 1.6. 迅速重叠

还有一件事情需要注意，当 view 重叠时（例如从开头走到了结束，或者反之），里面的内容会发生奇怪的移动...

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/CandidAntiqueBronco.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

解决方法就是将开头的内容连接到结束的位置，这样滚动起来就会流畅许多。

![在内容的结尾增加一个开头内容的副本](http://www.c4ios.com/images/cosmos/5/06.png)

然而，这个操作和 scrollview 无关。这个小技巧只依赖于你如何在结尾增加内容。

所以，回到 WorkSpace 文件中。

## 2. 正确的设置

增加一个新方法 `createIndicator()` ，用来给滚动视图创建和增加文本。

添加如下代码：

```swift
func createIndicator(text: String, at point: Point) {
    // 创建一个 TextShape
    let ts = TextShape(text: text)
    // 居中 TextShape
    ts.center = point
    // 添加到 canvas 上
    infiniteScrollView.add(ts)
}
```

在这个方法里，我们得到了一个 `String` 类型和一个 `Point` 类型，创建了一个新的文本形状的指示器，定位，然后添加到滚动视图里来。

太简单了。

现在，把 `addVisualIndicator` 的内容换成下放代码：

```swift
func addVisualIndicators() {
    // 设置指示器的最大数
    let count = 20
    
    // 设置每个指示器之间的间隔
    let gap = 150.0
    
    //对于每个指示器，设置开始时中心位置的偏移量
    let dx = 40.0
    
    // 初始化指示器的偏移量，因为我们要把指示器放在居中的位置
    let width = Double(count + 1) * gap + dx
    
    // 创建主要的指示器
    for x in 0...count {
        //给新的指示器创建中心点
        let point = Point(Double(x) * gap + dx, canvas.center.y)
        //创建新的指示器
        createIndicator("\(x)", at: point)
    }
    
    // 创建额外的指示器
    var x : Int = 0
    
    // 创建偏移量
    var offset = dx
    
    // 总宽度（包括无限滚动视图里最后一个 view，基于 width + screen width）
    // 所以，总宽度，和一个有多少个额外的指示器，在一定程度上是随机的
    // 这也是为什么我们要使用 while 循环
    
    // 当偏移量小于 view 的宽度时
    while offset < Double(infiniteScrollView.frame.size.width) {
        // 创建中心点，x 坐标值为 width + the current offset
        let point = Point(width + offset, canvas.center.y)
        // 创建宽度
        createIndicator("\(x)", at: point)
        // 对下一个点增加偏移量
        offset += gap
        // 增加 x 坐标值，作为下一个指示器的数值
        x += 1
    }
    
    // 更新 infiniteScrollView 的 contentSize
    infiniteScrollView.contentSize = CGSizeMake(CGFloat(width) + infiniteScrollView.frame.size.width, 0)
}
```

现在，运行之后一切都会流畅啦。

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/SelfreliantCoolHake.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

当数字滑动到结束时，就会无缝连接继续循环。

## 3. 打包

你已经将无限滚动视图添加到了 UIKit 的 `UIScrollView` 类里，这部分比较简单。真正有意思的是实现无限滚动的效果，这有赖于你是如何将内容添加进去的。

你已经学会了如何在滚动视图的结尾部分增加一点额外的内容，这样能让滚动的效果看起来更流畅、无缝连接。

在我们配置无限滚动视图时，大部分的时间都是在让内容显示在正确的位置上。正如这个例子所说明的一样，如果你想用代码让视图排列正确，可能需要使用一些小伎俩。

### 3.1. 代码

这里有 `InfiniteScrollView` 类的最终代码：

[InfiniteScrollView 代码](https://gist.github.com/C4Framework/60f9f8ec1824ce8a2807)