# Chapter 23 最后的组装

在这一章里，我们会把所有的组件组装在一起，创造一个优秀、统一、用户体验性极强的应用。

回顾一下，我们已经完成了下列事情：

1. 一个 UIScrollview 的子类，无限滚动视图
2. 星座创建类（Astrological Sign Provider），存储所有的符号数据
3. 星星背景，显示持续滚动的内容，有八个不同的层级
4. 非常棒的圆形菜单，具有各种复杂的动画效果和交互机制，有长按效果
5. 一个信息面板，链接到 C4 的网站
 
现在，想让 COSMOS 真正动起来，我们需要做下面几件事情：

1. 增加菜单、背景和信息面板
2. 通过选择行为和背景里的 `goTo` 方法将菜单和背景连接起来
3. 通过点击信息按钮把菜单和信息按钮连接起来
4. 增加一对环境音效音频文件，可以给应用提供背景音乐
 
那么，让我们开始吧。

## 1. WorkSpace

大部分的工作都是在 `WorkSpace` 里完成的，此外我们还需要微调一些类。WorkSpace 里的 `setup()` 方法是我们组合所有组件的主场。

如果你之前测试的时候修改过 WorkSpace，那么现在需要你清理一下代码，把背景颜色改为 `COSMOSbkgd`。

更改内容如下：

```swift
let COSMOSprpl = Color(red:0.565, green: 0.075, blue: 0.996, alpha: 1.0)
let COSMOSblue = Color(red: 0.094, green: 0.271, blue: 1.0, alpha: 1.0)
let COSMOSbkgd = Color(red: 0.078, green: 0.118, blue: 0.306, alpha: 1.0)

class WorkSpace: CanvasController {

    override func setup() {
        canvas.backgroundColor = COSMOSbkgd
    }
}
```

## 2. 星星

添加星星背景也是易如反掌，只要创建 `Stars` 副本，添加到 canvas 上，如下：

```swift
class WorkSpace: CanvasController {
    let stars = Stars()
    override func setup() {
        canvas.backgroundColor = COSMOSbkgd
        canvas.add(stars.canvas)  
    }
}
```

完成。运行，现在 COSMOS 可以滚动了。

## 3. 菜单

添加菜单的方法很简单，在添加完背景后添加菜单，如下面所示：

```swift
class WorkSpace: CanvasController {
    let stars = Stars()
    let menu = Menu()
    override func setup() {
        canvas.backgroundColor = COSMOSbkgd
        canvas.add(stars.canvas)  
        menu.canvas.center = canvas.center  
        canvas.add(menu.canvas) 
    }
}
```
### 3.1 菜单的 Frame

我们想让菜单的表现如下：只有在用户点击菜单时才会打开菜单，例如，不能点击屏幕的其他地方。这也是我们把 canvas 设置的很小的原因（只有 80*80）。由于 frame 这么小，菜单的交互就限制在 80*80 的界面上。

就是这样。

## 4. 信息面板

这一步小事一桩，给信息面包创建另外一个变量，接着在 `setup()` 方法里实例化，先添加到菜单上，再添加到 WorkSpace 的 canvas 上。 

回到 `InfoPanel.swift`，在 `setup()` 里增加下列方法：

```swift
self.canvas.opacity = 0.0
```

> 因为不透明度是 `0.0`，canvas 不会接收任何点点击行为，所以添加到菜单上是可行的，用户能够点击菜单不受影响。

现在你的 WorkSpace 应该是下面这个样子：

```swift
class WorkSpace: CanvasController {
    let stars = Stars()
    let menu = Menu()
    let info = InfoPanel()

    override func setup() {
        canvas.backgroundColor = COSMOSbkgd

        canvas.add(stars.canvas)

        menu.canvas.center = canvas.center
        canvas.add(menu.canvas)

        canvas.add(info.canvas)
    }
}
```

## 5. 关联菜单和背景

想要关联菜单和背景，我们需要使用一点小窍门：给菜单增加一个特殊的行为，连到背景里的 `goTo()` 方法里。

回到 `Menu.swift`文件，在文件的最上面，也就是声明类的上面，增加下列代码：

```swift
typealias SelectionAction = (selection: Int) -> Void
```

这是类型别名，能让我从另一个类里制定方法。选择行为实际上就是我们想发送给菜单的当前行为。这句话翻译的有问题，重新来一次。

创建变量来保存选择行为的引用：

```swift
var selectionAction : SelectionAction?
```

回到 `createGesture()`，在 `case .Cancelled:` 闭包里增加下列代码：

```swift
if let sa = self.selectionAction  where self.menuSelector.currentSelection >= 0 {
    sa(selection: self.menuSelector.currentSelection)
}
```

> 因为我们有时候需要把当前的选项设置为 -1，因此需要额外的 `where` 表达式。

回忆一下第 14 章的这个方法：

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

这个方法接收的参数和我们刚才创建的别名 `SelectionAction` 一样。它接收一个整数，找到目标对象并滚动到对应的位置。

最后，回到 WorkSpace，增加下列代码：

```swift
menu.selectionAction = stars.goto
```

WorkSpace 里的代码应该是下面这样：

```swift
class WorkSpace: CanvasController {
    var background : ParallaxBackground?
    var menu : Menu?
    var info : InfoPanel?

    override func setup() {
        canvas.backgroundColor = COSMOSbkgd

        background = ParallaxBackground()
        canvas.add(background?.canvas)

        menu = Menu()
        menu?.canvas.center = canvas.center
        canvas.add(menu?.canvas)

        info = InfoPanel()
        canvas.add(info?.canvas)

        menu?.selectionAction = background?.goto
    }
}
```

运行，效果如下：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/WetJoyfulHarborseal.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 6. 连接菜单和信息面板

连接菜单和信息面板的步骤几乎和之前一样。

在 `Menu.swift` 文件里，在最上面也就是在所有类之前，增加下列代码：

```swift
typealias InfoAction = () -> Void
```

同样，在 `Menu.swift` 中增加下列类变量：

```swift
var infoAction : InfoAction?
```

接着，找到菜单手势里的 `.Cnacelled` 部分，增加下列代码：

```swift
if let ib = self.menuSelector.infoButton {
    if ib.hitTest(center, from: self.canvas),
        let ia = self.infoAction {
            delay(0.75) {
                ia()
            }
    }
}
```

最后，回到 WorkSpace，增加下列代码

```swift
menu.infoAction = info.show
```

运行，效果如下：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/EthicalIdealDevilfish.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 7. 环境音效

工程里已有两个音频文件，在应用启动后会立即播放音频。

在 WorkSpace，创建下面两个变量：

```swift
let audio1 = AudioPlayer("audio1.mp3")!
let audio2 = AudioPlayer("audio2.mp3")!
```

在 `setup()`，循环播放，如下：

```swift
audio1.loops = true
audio1.play()

audio2.loops = true
audio2.play()
```

听起来像是在外太空了。

## 8. 最初的星座偏移量

我们倾向选择让星星从特定的某个星座开始偏移，打开 `Stars.swift`，在 `setup()` 方法底部增加下列方法：

```swift
bigStars.contentOffset = CGPointMake(view.frame.size.width * CGFloat(gapBetweenSigns / 2.0), 0) 

```

## 9. 最后一步操作

如果下面的操作你还没设置过，那么现在来设置吧。

点击 Xcode 上边的 COSMOS 工程，在 General 分页下，找到 Deployment Info 部分，选择下列设置：

 - Portrait: 勾选
 - Upside Down: 勾选
 - Landscape left: 不勾选
 - Landscape right: 不勾选
 - Hide Status Bar: 勾选
 - Requires Full Screen: 勾选

![](http://www.c4ios.com/images/cosmos/23/01.png)

终终终终终

于

完

成

了。