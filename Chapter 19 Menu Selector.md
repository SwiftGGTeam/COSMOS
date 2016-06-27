# Chapter 19 菜单选择器

现在我们开始实现菜单的选择器。为了测试功能是否正确，最简单的办法就是让选中的菜单消失。为了实现这个目标，修改工程的 `WorkSpace`，添加如下的 `setup()`方法：

```swift
override func setup() {
    canvas.backgroundColor = COSMOSbkgd
    
    let rings = MenuRings()
    rings.canvas.center = canvas.center
    canvas.add(rings.canvas)
    rings.revealDashedRings?.animate()
    rings.revealHideDividingLines(1.0)
    rings.thickRingOut?.animate()
    rings.thinRingsOut?.animate()
    
    let icons = MenuIcons()
    icons.canvas.center = canvas.center
    canvas.add(icons.canvas)
    icons.signIconsOut?.animate()
    icons.revealSignIcons?.animate()
}
```

运行 app，你会看到如下画面：

![](http://www.c4ios.com/images/cosmos/19/01.png)

OK, 本章的目标就是实现当用户在菜单栏上进行拖拽时，菜单选择器会相对应的出现在用户当前手指的位置。像下面这样：

![](https://zippy.gfycat.com/VainTautKagu.mp4)

我们需要组合以下的几个任务来实现这个效果：

- 一个用来跟踪用户触摸位置的长按手势
- 一个用来把触摸的位置转换为在菜单上的位置和角度的方法
- 一个用来表示选择器选中/取消选中两种状态的形状。

## 1. 添加手势

第一步就是把手势附加到画布上，并开始跟踪用户触摸的位置。

打开 `MenuSelector.swift`，并添加如下两个方法：

```swift
func createGesture() {
    canvas.addLongPressGestureRecognizer { (locations, center, state) -> () in
        switch state {
        case .Changed:
            self.update(center)
        default:
            _ = ""
        }
    }
}

func update(location: Point) {
    print(location)
}
```

然后，修改 `setup()` 为下面这样：

```swift
public override func setup() {
    canvas.frame = Rect(0,0,80,80)
    canvas.backgroundColor = C4Pink
    createGesture()
}
```

这个时候，回到工程的 `WorkSpace` 并添加如下的代码到 `setup()` 方法：

```swift
class WorkSpace: CanvasController {
    override func setup() {
      canvas.backgroundColor = COSMOSbkgd
        
        let rings = MenuRings()
        rings.canvas.center = canvas.center
        canvas.add(rings.canvas)
        rings.revealDashedRings?.animate()
        rings.revealHideDividingLines(1.0)
        rings.thickRingOut?.animate()
        rings.thinRingsOut?.animate()
        
        let icons = MenuIcons()
        icons.canvas.center = canvas.center
        canvas.add(icons.canvas)
        icons.signIconsOut?.animate()
        icons.revealSignIcons?.animate()
        
        let selector = MenuSelector()
        selector.canvas.center = canvas.center
        canvas.add(selector.canvas)
    }
}
```

运行一下，你应该能够看到下面这样的效果：

![](https://zippy.gfycat.com/FaithfulOrganicBigmouthbass.mp4)


现在你从粉红色的块开始长按，并拖动，你会发现你触摸的位置会被持续打印在终端上。

![](http://c4ios.com/cosmos/02.png)


我们使用 `switch` 语句来判断触摸的位置是否有更新，但默认的情况下这个方法什么都不做。现在我们仅仅只是测试了手势的拖拽，最终我们还会跟踪手势开始和结束的状态。

## 2. 计算角度

我们要知道从画布的中心到用户触摸的位置的角度，当计算点之间的角度的时候。下面三条很重要：

1. 计算一个角度总是默认用最小的非负角（我们需要针对此调整一下）。下面两种状态返回同样的值：

![](http://c4ios.com/cosmos/03.png)

2. 我们使用像右边这样的以顺时针方向旋转的坐标系。

![](http://c4ios.com/cosmos/04.png)

3. 我们需要三个点来计算用户触摸点和画布中点的角度。这三个点分别是：a）中点右边，x 轴上的任意一个点；b）中心点；c）用户触摸的点。有了这三个点我们就能计算出角度：

![](http://c4ios.com/cosmos/05.png)

替换函数 `update(location:)` 的内容为如下代码：

```swift
let a = Vector(x:self.canvas.width / 2.0+1.0, y:self.canvas.height/2.0)
let b = Vector(x:self.canvas.width / 2.0, y:self.canvas.height/2.0)
let c = Vector(x:location.x, y:location.y)

var ϴ = c.angleTo(a, basedOn: b)
print(ϴ)
```

运行项目。注意当触摸的位置在屏幕的左边穿过 x 轴的过程中，我们的值先变为 𝜋 再减小为 0。我们可以按下面这样来调整。

```swift
if c.y < a.y {
    ϴ = 2*M_PI - ϴ
}
```

> 把这块代码放在 `print(ϴ)` 的前面。

现在我们要把角度转换为菜单的序号。我们菜单有 12 项，总的角度是 360°。 所以我们只需要：

```swift
let index = Int(radToDeg(ϴ)) / 30
```

现在我们已经知道了现在用户触摸的点是哪一个菜单项了。

最后，修改我们的打印语句：

```swift
print(index)
```

执行代码，你会发现终端已经输出了 0 到 11 的值，而不是之前的角度的值。


## 3. 实现高亮

高亮的设计就是刚好覆盖在被选中的图标上，它有一个奇怪的形状。

![](http://c4ios.com/cosmos/06.png)

我们可以用具体的弧度和维度来实现一个基本形状，然后旋转这个形状来匹配不同的菜单图标。但是，我还是想实现得简单一些。

首先，我们用一个楔状物（wedge）来定义高亮区。楔状物代表的是圆中的一个扇形图，所以它的外边缘已经是圆的了。我们来新建一个楔形并把它加入到菜单中。

添加如下代码到类中：

```swift
var highlight : Shape!
```

然后添加如下的方法：

```swift
func createHighlight() {
    highlight = Wedge(center: canvas.center, radius: 156, start: M_PI/6.0, end: 0.0, clockwise: false)
    highlight.fillColor = COSMOSblue
    highlight.lineWidth = 0.0
    highlight.opacity = 0.8
    highlight.interactionEnabled = false
    highlight.anchorPoint = Point()
    highlight.center = canvas.center
    
    canvas.add(highlight)
}

```

最后，在 `setup()` 中调用这个函数：

```swift
public override func setup() {
    canvas.frame = Rect(0,0,80,80)
    canvas.backgroundColor = C4Pink
    createGesture()
    createHighlight()
}
```

运行一下，你会看到：

![](http://c4ios.com/cosmos/07.png)

我们刚才做的就是创建了一个楔子并把它放到了菜单的第一个位置（也就是序号 0 的位置）。它的基准点被设置为左上角，之后我们旋转这个 view 的时候也是围绕此点来旋转的。剩下的代码是一些必要的布局。

### 3.1. 变换楔形

当楔形放在图标上时，我们要把楔形在图标之外的部分去掉。最简单的办法就是给楔形做一个遮罩。

添加如下代码到 `createHighlight()` 函数中，把楔形添加到画布的操作之前。

```swift
let donut = Circle(center: highlight.center, radius: 156-54/2.0)
donut.fillColor = clear
donut.lineWidth = 54
highlight.mask = donut
```

运行：

![](http://c4ios.com/cosmos/08.png)

哇哦~

这创建了一个直径一直到图标中心点的圆，而且线宽填充了整个区域。用这个形状来当做遮罩使得不透明的地方都能展示，并且隐藏了透明的部分。

下面是把这个遮罩添加到楔形的效果：

![](http://c4ios.com/cosmos/09.png)

![](http://c4ios.com/cosmos/10.png)


### 3.2. 移动它

现在，我们要把高亮区的位置和我们的手势关联起来。

在 `update(location:):` 函数的末尾添加如下的代码：

```swift
let rotation = Transform.makeRotation(degToRad(Double(index) * 30.0), axis: Vector(x: 0,y: 0,z: -1))
highlight.transform = rotation 

```

运行一下：

![](https://zippy.gfycat.com/SnivelingFlamboyantFieldspaniel.mp4)


## 4. 一些行为

我们需要楔形的行为遵从以下的规则：

- 只有当用户触摸的位置移动到新的图标时，才触发位置的改变。
- 每次只高亮一个图标，也就是用户的手指的位置的图标
- 当用户的触摸离开一个图标后，就取消这个图标高亮的效果。

创建一个类变量来存储当前的选择，比如这样：

```swift
var currentSelection = -1

```

然后，修改 `update()` 函数，添加判断当前的图标是否有改变的逻辑：

```swift
if currentSelection != index {
    currentSelection = index
    let rotation = Transform.makeRotation(degToRad(Double(currentSelection) * 30.0), axis: Vector(x: 0,y: 0,z: -1))
    highlight.transform = rotation
}
```

这在界面上看不出来有什么作用，但相信我：这样做可以避免频繁更新高亮区，只在位置变化时才更新。

然后，把用户触摸点到画布中心的距离判断包裹在一个 `if-else` 语句块中。在这里，只要用户触摸的点离中心的距离在 `102` 到 `105` 之间，我们都认为触摸是在图标的区域。

`update()` 函数的结束应该是这样：

```swift
let dist = distance(location, rhs: self.canvas.bounds.center)

if dist > 102 && dist < 156 {
    if currentSelection != index {
        currentSelection = index
        let rotation = Transform.makeRotation(degToRad(Double(currentSelection) * 30.0), axis: Vector(x: 0,y: 0,z: -1))
        highlight.transform = rotation
    }
}
```

此时运行程序，会发现高亮区只有在触摸在图标上的时候才会更新：

![](https://zippy.gfycat.com/CapitalCarefreeIceblueredtopzebra.mp4)


最后，添加下面的代码到 `createHighlight()`

```swift
highlight.hidden = true

```


以及，修改之前的 if 语句块，添加 else 分支来控制高亮区的显示和消失：

```swift
if dist > 102 && dist < 156 {
    highlight.hidden = false
    if currentSelection != index {
        currentSelection = index
        let rotation = Transform.makeRotation(degToRad(Double(currentSelection) * 30.0), axis: Vector(x: 0,y: 0,z: -1))
        highlight.transform = rotation
    }
} else {
    highlight.hidden = true
}
```

如果用户触摸在图标内部的区域我们就通过把高亮区的 hidden 属性设置为 `false` 来显示它， 反之则隐藏。

最后，同样重要的是，在用户的手指离开屏幕的时候我们需要隐藏高亮区。为了实现这个功能我们需要添加一个逻辑：

```swift
case .Cancelled, .Ended, .Failed:
    currentSelection = -1
    highlight.hidden = true
```

> `currentSelection = -1` 在这里是因为我们希望当用户完成拖拽之后释放当前的选中的菜单。

这里的逻辑是，当手势被取消，或者手势失败（由设备控制的状态），或者手势结束的时候，我们检查高亮区是否正在显示，如果是则关闭它。

整个方法应该长这样：

```swift
func createGesture() {
    // 为菜单选择添加一个长按的手势
    canvas.addLongPressGestureRecognizer { (locations, center, state) -> () in
        switch state {
        case .Changed:
            self.update(center)
        case .Cancelled, .Ended, .Failed:
            self.currentSelection = -1
      self.highlight.hidden == true
        default:
            _ = ""
        }
    }
}
```

此时运行程序：


![](https://zippy.gfycat.com/NeighboringAgonizingAztecant.mp4)

## 5. 菜单的标签

菜单的标签出现在菜单的中间，展示的是当前用户所选的星象的名称。

创建一个类变量：

```swift
var menuLabel : TextShape!
```

添加如下的方法：

```swift
func createLabel() {
    let f = Font(name: "Menlo-Regular", size: 13)!
    menuLabel = TextShape(text: "COSMOS", font: f)!
    menuLabel.center = canvas.center
    menuLabel.fillColor = white
    menuLabel.interactionEnabled = false
    canvas.add(menuLabel)
    menuLabel.hidden = true
}
```

在 `setup()` 中添加该方法的调用。

这时，在 `update(location:)` 中的 `highlight.hidden = false` 上方添加：

```swift
menuLabel.hidden = false
```

在 `if` 语句中，添加如下的逻辑：

```swift
ShapeLayer.disableActions = true
menuLabel?.text = AstrologicalSignProvider.sharedInstance.order[index].capitalizedString
menuLabel?.center = canvas.bounds.center
ShapeLayer.disableActions = false
```

上述代码实现了当 Core Animation 的动画因为我们改变了形状的路径而无效时，更新标签上的文字。

这时，在同一个方法的 `else` 块里，隐藏标签：

```
menuLabel.hidden = true

```

你还需要添加 `hidden = true` 这一行到长按事件处理方法的 `.Cancelled` 分支。

现在，你的 `update(location:)` 方法应该是这样的：

```swift
func update(location: Point) {
    let a = Vector(x:self.canvas.width / 2.0+1.0, y:self.canvas.height/2.0)
    let b = Vector(x:self.canvas.width / 2.0, y:self.canvas.height/2.0)
    let c = Vector(x:location.x, y:location.y)
    
    var ϴ = c.angleTo(a, basedOn: b)
    
    if c.y < a.y {
        ϴ = 2*M_PI - ϴ
    }
    
    let index = Int(radToDeg(ϴ)) / 30
    
    let dist = distance(location, rhs: self.canvas.bounds.center)

    if dist > 102 && dist < 156 {
        menuLabel.hidden = false
        highlight.hidden = false
        if currentSelection != index {
            ShapeLayer.disableActions = true
            menuLabel?.text = AstrologicalSignProvider.sharedInstance.order[index].capitalizedString
            menuLabel?.center = canvas.bounds.center
            ShapeLayer.disableActions = false
            
            currentSelection = index
            let rotation = Transform.makeRotation(degToRad(Double(currentSelection) * 30.0), axis: Vector(x: 0,y: 0,z: -1))
            highlight.transform = rotation
        }
    } else {
        highlight.hidden = true
        menuLabel.hidden = true
        currentSelection = -1
    }
}
```

而 `setup()` 方法应该是这样：

```swift
public override func setup() {
    canvas.frame = Rect(0,0,80,80)
    canvas.backgroundColor = C4Pink
    createGesture()
    createHighlight()
    createLabel()
}
```

运行一下程序，行为应该是这样的：

![](https://zippy.gfycat.com/NiceInfiniteBlueshark.mp4)

## 6. 信息按钮

在程序里我们设置了一个信息页面包括一些对 app 的介绍以及一个跳转到 C4 网站的链接。通过一个按钮来从菜单页面跳转到信息页面。

添加类变量：

```swift
var infoButton : View!

```

稍后我们要做点有趣的操作，所以用 view 来做按钮。

首先我们要采用 iOS 中标准的信息图案来作为背景图，像这样：

![](http://c4ios.com/cosmos/11.png)

这个图片是之前我在另外一个工程里通过如下代码抓取到的：

```swift
let button = UIButton(type: UIButtonType.InfoLight)
button.imageView?.tintColor = UIColor.whiteColor()
let img = button.imageForState(.Normal)

let fileManager = NSFileManager.defaultManager()
let data = UIImagePNGRepresentation(img!)
fileManager.createFileAtPath("/Users/travis/Desktop/info.png", contents: data, attributes: nil)

```

这段代码可以生成一个 png 图像文件，之后在 PhotoShop 里打开它，修改颜色为白色，并导出 3x 的版本。

iOS 标准的信息按钮的大小是 22x22 像素，很小。如果我们真的做一个这样小的按钮，会很难点击。

通过 `view` 来实现 `infoButton` 让我们可以实现比实际显示的图像更大的点击区域。

拷贝如下的代码到你的类中：

```swift
func createInfoButton() {
    infoButton = View(frame: Rect(0,0,44,44))
    let buttonImage = Image("info")!
    buttonImage.interactionEnabled = false
    buttonImage.center = infoButton.center
    infoButton.add(buttonImage)
    infoButton.opacity = 1.0
    infoButton.center = Point(canvas.center.x, canvas.center.y+190.0)
    canvas.add(infoButton)
}
```

代码的逻辑很直接：生成一个小图像，以及一个大一些的 view。把图像添加到 view 中，然后把 view 添加到画布里并隐藏。

下一步，添加两个变量和一个方法来实现动画得隐藏/显示按钮：


```swift
var revealInfoButton : ViewAnimation?
var hideInfoButton : ViewAnimation?

func createInfoButtonAnimations() {
  revealInfoButton = ViewAnimation(duration:0.33) {
    self.infoButton?.opacity = 1.0
  }
  revealInfoButton?.curve = .EaseOut

  hideInfoButton = ViewAnimation(duration:0.33) {
    self.infoButton?.opacity = 0.0
  }
  hideInfoButton?.curve = .EaseOut
}

```

我们现在不用太关注这些动画的实现，下一章会具体讲解。

最后，我们要检查用户是否按下信息按钮，因为信息按钮是在图标菜单之外的，添加如下的代码到 `update` 的 `else` 块中：

```swift
if let l = infoButton  {
    if l.hitTest(location, from:canvas) {
        menuLabel.hidden = false
        ShapeLayer.disableActions = true
        menuLabel.text = "Info"
        menuLabel.center = canvas.bounds.center
        ShapeLayer.disableActions = false
    }
}
```

## 7. 菜单声音

选择菜单的时候能够发出一些声音。最终，选择器会和打开和关闭菜单同步，所以我们这里需要处理三件事情：

1. 当用户手指移动到新的菜单时有一个滴答声
2. 菜单打开的声音
3. 菜单关闭的声音

添加如下的三个变量到类里：

```swift
let tick = AudioPlayer("tick.mp3")!
```

在 `setup` 中添加：

```swift
tick.volume = 0.4

```

然后，在 `if currentSelection != index {...}` 的右边添加：

```swift
tick.stop()
tick.play()
```

我们调用 `stop()` 的原因是 Core Audio 在默认情况下在播放声音之前会等待上一次播放结束。在播放前结束声音可以消除当用户在菜单上快速移动时的圣印播放的延迟。

运行程序，现在应该已经可以听到不错的声音效果了。

## 8. 清理

现在，修改 `setup()` 为：

```swift
public override func setup() {
    // 创建 frame，和其他类的代码一样少
    canvas.frame = Rect(0,0,80,80)
    canvas.backgroundColor = clear
    createHighlight()
    createLabel()
    createInfoButton()
    createInfoButtonAnimations()
    tick.volume = 0.4
}
```

同时，也清除 `setup` 中一些不需要的东西。

最终，我们会在别的地方创建手势识别器，然后把它从这个类中移除。

你可以查看当前版本的 [MenuSelector.swift](https://gist.github.com/C4Framework/fc16b1e3552a9de94955) 文件。

深呼吸。

你做的不错。

就快结束了。