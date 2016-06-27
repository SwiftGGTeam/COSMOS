# Chapter 2 设计 COSMOS

3 月 11 日，讨论后我获得了一个可以在知名网站上发布一篇教程的机会。所以，我开始和 Jake 讨论一些能够设计、开发、发布的概念，能够抓住 C4 Swift 版本的要点，学习如何使用这个新系统来创建动画...我们想到了：会有很多基础的动画出现，然后组合成一个优雅的界面。之后我们看了很多 UI 视频，然后进行头脑风暴。

Jake 想到了一个点子：

> ...中间出现了一个实心圆圈，当你点击后，它会稍微缩小一下（原来的 90%大小），然后从中心接着放射出八个圆圈，每个圆圈都比之前的圆圈大一点，最外层的圆圈里有不同的图标。点击显示在中间的叉号关闭所有的圆圈，回到初始状态。 

![](http://www.c4ios.com/images/cosmos/2/01.png)

于是我们看了更多的视频。

讨论这个概念。

好像行得通。

开始围绕这个计划工作吧。

这就是我们的工作方式。

## 1. 模拟和测试

实际的应用比较简单，尽管在界面和背景中有很多细微的组件需要花时间调整。此外由于整个应用称得上简单——亦可算复杂——这个从头到尾构建的过程能给我们的教程提供最好的素材。

![](http://www.c4ios.com/images/cosmos/2/02.png)

Jake 最终将他的设计稿变成了一个单页面的小程序，里面有一个炫酷的动画菜单和一个容纳所有内容的多层视差背景。我看了一下，思考如何才能让两个组件合并起来。

## 2. 背景

背景部分的工作比较容易分解，主要就是很多不同内容的图层在按照不同的速度移动。

![](http://www.c4ios.com/images/cosmos/2/03.png)

里面有：
 
* 大星星
* 小星星
* 连接星星的线
* 三个背景星星层
* 两个星云层

这样完全可以做到。所以在和 Jake 沟通之后，我写了一个清单列出需要他定义的一些东西：

* 角度/指示器动画
* 单个的星座
* 三层的前景风格 + 运动效果
* 三个背景星星层的风格 + 运动效果（包括上面的星星）
* 两个星云背景层的风格 + 运动效果
 
第一步，得到 layer 的数量，同时获取视差角度...需要八个，所以我先用 10 个来测试一下实际的效果。
 
> 本章的代码只是我在真正开发之前的一些测试展示效果，所以当你看完这章后，记得删掉在本章添加的所有代码。

```swift
class WorkSpace: CanvasController {
    // 创建一个空的数组变量，用来添加 layers
    var layers = [UIScrollView]()

    override func setup() {
        // 当 layer 数量小于 10 时，执行循环体里的代码
        repeat {
            // 创建一个 layer，它的 frame 值和 canvas 的 frame 值一样
            let layer = UIScrollView(frame: view.frame)
            // 设置每层 layer 内容的大小，高度为 0 ，防止屏幕垂直滚动
            layer.contentSize = CGSizeMake(layer.frame.size.width * 10, 0)
            // 把 layer 添加到 canvas 上以及数组里
            canvas.add(layer)
            layers.append(layer)
        } while layers.count < 10
    }
}
```

很直观吧。使用的工程的文件正是前一章中提到的，我在 WorkSpace 文件中添加一个 repeat 循环体，来创建新的 layer 添加到 canvas 上，直到创建完第 10 个 layer。每创建一个 layer，我都会把 layer 的 contentSize 设得超级大（在这个例子中，是 canvas 的十倍宽）（校者注：原文是二十倍宽，应该是勘误）。设置 contentSize 的高度为 0，这样就不会垂直滚动了。

在这时，如果我运行应用，我会什么都看不到，所以我修改一下循环体里代码，给每个 layer 增加一个 label 控件。

```swift
class WorkSpace: CanvasController {
    // 创建空的数组变量，用来存储 layer
    var layers = [InfiniteScrollView]()

    override func setup() {
        // 当 layer 数量小于 10时，执行循环体里的代码
        repeat {
            // 创建一个 layer，它的 frame 值和 canvas 的 frame 值一样
            let layer = InfiniteScrollView(frame: view.frame)
            // 设置每层 layer 内容的大小，高度为 0 ，防止屏幕垂直滚动
            layer.contentSize = CGSizeMake(layer.frame.size.width * 10, 0)
            // 把 layer 添加到 canvas 上以及数组里
            canvas.add(layer)
            layers.append(layer)

            // 创建一个中心点变量，用来定位这些 label
            var center = Point(24,canvas.height/2.0)
            // 计算 layer 的数量（因为我们要加最后一个 layer，从 10 开始倒序添加）
            let layerNumber = 10 - layers.count
            // 创建字体，字号是当前 layer 的数量
            let font = Font(name: "AvenirNext-DemiBold", size:Double(layers.count+1) * 8.0)!
            // 创建运行循环体知道每个 layer 都有一个 label
            repeat {
                // 创建一个 label
                let label = TextShape(text: "\(layerNumber)", font: font)!
                // 居中
                label.center = center
                // 更新中心点的位置
                center.x += 130.0
                // 把 layer 添加到数组里
                layer.add(label)
            } while center.x < Double(layer.contentSize.width)
        } while layers.count < 10
    }
}
```

修改原来的设置，加入一个内嵌的 repeat 循环体，直到全部 layer 的包含一个 label —— 每个 label 基于所在的 layer 编号。

现在运行程序，应用里会出现 label 控件，不过！如果我滚动界面，只有一个 layer 在滚动...

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/DishonestDescriptiveBassethound.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

下一步就是创建一个观察器，查看一下最上层的 layer，在滚动时将剩下的 layer 移走。在 setup 的最下方，添加下列代码：

```swift
if let top = layers.last {
    // 创建一个变量
    var c = 0
    // 添加 WorkSpace 作为最上层 layer 的 contentOffset 的观察者
    top.addObserver(self, forKeyPath: "contentOffset", options: NSKeyValueObservingOptions.New, context: &c)
}
```

这一步把 WorkSpace 作为最上层 layer 的 contentOffset 的观察者。现在，让代码更漂亮一些，我创建一个函数，关联 layer 的运动轨迹，改变其他 layer 的轨迹，如下：

```swift
override func observeValueForKeyPath(keyPath: String?, ofObject object: AnyObject?, change: [String : AnyObject]?, context: UnsafeMutablePointer<Void>) {
    // 遍历所有的 layer，停在在从上数第二层的 layer 那里
    for i in 0..<layers.count-1 {
        // 获取当前的 layer
        let layer = self.layers[i]
        // 基于 layer 的位置创建一个 mod 值（layer 0 = 0.1, layer 1 = 0.2, ...）
        let mod = 0.1 * CGFloat(i+1)
        // 获取最顶层 layer 偏移量的 x 值
        if let x = layers.last?.contentOffset.x {
            // 设置内容的偏移量是当前 layer * mod
            layer.contentOffset = CGPointMake(x*mod,0)
        }
    }
}
```

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/RequiredSilentAfricangoldencat.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

漂亮。现在我们知道这 10 个 layer 能够成功工作了...不过，如果当图片放在这些层上会发生什么呢？...是时候测试一下了。Jake 大致估计了一下，每层星星的数量大约在 15 个，还给我一个小的白星星素材。

我接着把内部 repeat 循环里的 label 换成图片，如下：

```swift
// 不需要中心位置了，记得每个 layer 有 10 * 15 个星星
let starCount = layers.count * 15
canvas.backgroundColor = black
// 循环，直到 starCount
for _ in 0..<starCount {
    // 给每个星星创建一张图片
    let img = Image("6smallStar")!
    // 允许图片可以适当按比例缩放
    img.constrainsProportions = true
    // 缩放图片的宽度
    img.width *= 0.1 * Double(layers.count+1)
    // 将中心点设置为 layer 随便某个位置上
    img.center = Point(Double(layer.contentSize.width)*random01(),canvas.height*random01())
    // 添加到数组中
    layer.add(img)
}
```

运行程序，模拟器中应用效果如下：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/NippyQuaintBrownbear.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

应用在 iPhone 5 上运行良好。这 10 层的运行结果非常正常，那么剩下的问题就是界面的美观程度了。这时，Jake 也基本上制定了在这一节一开始提到的列表中的全部细节。

### 2.1 单个的星座

12 个星座的符号由三部分构成：大星星、小星星和线。Jake 用下图记录每种星座中星星的位置：

![注意观察大星星和小星星](http://www.c4ios.com/images/cosmos/2/09.png)

### 2.2 三层的前景风格 + 运动效果

接下来定义三个前景的 layer 中的星星该怎么样运动。Jake 的想法是将这些星星移动到合适的位置。所以我们决定使用三层不同的 layer 来代表大星星、小星星和线。当应用中移动到某个特定的符号出现的位置时，当前的星星需要出现在正确的位置上。接着，只要当视图开始移动改变时，这儿需要有一个非常短的来改变连接星星之间线的位置的动画。

![](http://www.c4ios.com/images/cosmos/2/10.png)

### 2.3 三个背景星星层的风格 + 运动效果（包括上面的星星）

接下来需要定义背景里的星星会以什么样的速度在移动。非常简单，Jake 认为从上往下这三层每一层的移动速度应该是最顶层的 5%、15%、20%。他也同时对每层有多少个星星有了一个大概的猜测。

![](http://www.c4ios.com/images/cosmos/2/11.png)

### 2.4 两个星云背景层的风格 + 运动效果

接下来，Jake 定义了星云和光晕的风格以及移动的效果。这一步甚至比上一步还要简单，因为光晕几乎不会移动，而星云层大约是最顶层 10% 的速度。

![](http://www.c4ios.com/images/cosmos/2/12.png)

### 2.5 角度/指示器动画

最后一步，整个窗口的底部会有一条虚线，每一划间隔 20 个单位。之后，当单个星座出现在屏幕中央时，下面出现一根较长一点的白线和星座所代表的符号，以及会显示星座出现的位置（按度测量）。

![最初的图层说明，我们后来不断对图层进行了更改](http://www.c4ios.com/images/cosmos/2/13.png)

### 2.6 最后

在继续接下来的工作前要干的最后一件事，就是写一个清单列出即将要开发的不同的 layer。Jake 实在是太好了，都给我发图解释清楚了：

![](http://www.c4ios.com/images/cosmos/2/14.png)

## 3. 菜单

菜单「看起来」挺简单，实际上不然。其实，唯一需要我搞懂的就是如何为这些星座符号做出动画效果。

![Jake 想用这些符号作为星座的参考](http://www.c4ios.com/images/cosmos/2/15.png)

实际上，给它们添加动画效果这事简单，难的地方在于如何做这些图形。我们希望它们有自己的贝塞尔路径，但是创建的过程确实十分痛苦，因为我们不知道他们的路径点，而且像是 IllUstrator 这样的软件也无法提供位置数据；还有，我不想写一个 SVG 导入工具，那也太多余了...

那么，我们该怎么办呢？

使用 [PaintCode](http://www.paintcodeapp.com/) 画出外形，接着添加曲线轨迹，保存到 Core Graphics 代码里，如下：

```objective-c
UIBezierPath* bezier2Path = UIBezierPath.bezierPath;
[bezier2Path moveToPoint: CGPointMake(250, 200)];
[bezier2Path addLineToPoint: CGPointMake(150, 200)];
[bezier2Path addCurveToPoint: CGPointMake(100, 150) controlPoint1: CGPointMake(122.4, 200) controlPoint2: CGPointMake(100, 177.6)];
...
[bezier2Path closePath];
```

当我把上面的 Objective-C 代码翻译成下面的 Swift 代码后：

```swift
let bezier = Path()

bezier.moveToPoint(Point(250,200))
bezier.addLineToPoint(Point(150,200))
bezier.addCurveToPoint(Point(100,150), control1:Point(122.4,200), control2:Point(100,177.6))
...
```

事情开始变得更清晰、容易处理了。既然我已经将星座符号的外形添加到了 C4 代码中，也无需费太多力就能实现我们想要实现的动画效果。

比如，让 shape 渲染全部的路径，声明：

```swift
shape.strokeEnd = 1.0
```

### 3.1 标注准则

到了这一步，我准备创建菜单界面了。因此需要下面的标注准则，用于确定菜单上所有元素的具体的位置、尺寸等等。

Jake 的工作做的真棒，给我准备了这张图：

![为码农做菜单界面提供的所有细心细节](http://www.c4ios.com/images/cosmos/2/16.png)

## 4. 该进行下一章了

基本的视觉概念都解释了，现在该做一些实际的开发工作了。不过，在这之前，我需要总结一些接下来会遇到的问题：
  
1. 自定义形状 - 在应用中我会复用很多形状，也会给它们添加动画效果。因此我会用自定义的贝塞尔曲线路径，而不是单单导入图片资源。
2. 复杂的动画序列 - 会有非常复杂的动画序列和调速，直到得到正确的菜单显现外展内收的效果。
3. 定义手势交互 - 我想让手势交互越简单越好，当然了还要让它们独一无二。
4. 视差 + 无限滚动视图 - 必须给应用增加视差效果，所以需要非常小心的处理。开发的同时也要保证应用的高性能。

> 记得删除掉 WorkSpace.swift 文件里的测试代码...只有一个空的 setup() 方法。
 
继续下一章吧！