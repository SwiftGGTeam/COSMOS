# Chapter 18 图标菜单

接下来，要把图标添加到菜单里，然后实现动画效果。图标的动画非常直接：一开始像是一个点，慢慢变大，移动到最终的位置，出现完整的形状，如下图：

![](http://www.c4ios.com/images/cosmos/18/01.png)

## 1. 定位

这部分复杂的地方不是动画和移动效果，这些都简单，真正复杂的是如何计算每个形状的位置，让一个点慢慢转变成完整的形状。有一些方法可以实现这种效果：

1. 创建一个点和一个形状（shape），一直隐藏形状，等待点移动到自己的位置之后隐藏点，显示形状。
2. 创建一个点，让这个点成为形状的一部分，移动点到某个位置，然后把点转变成完全的形状。
3. 有一个点，但不是真的点，只是看起来像是一个点。把形状的 `strokeEnd` 设置到正确的位置上，形状的 `lineCap` 是 `.Round`。
 
我们使用第三种方法 `strokeEnd` 来实现图标的动画。然而，问题出现了，每个形状的“初始”点各不同 —— 我们需要知道每个形状在点和全部出现这两种情况下的偏移量。

下面是摩羯座的图标，起始点已经高亮标注出了：

![](http://www.c4ios.com/images/cosmos/18/02.png)

起始点的位置是 `{30.0,12.2}`，对应的 frame 是 `{0.750,0.387}`。

创建一个「点」的效果，我们只需要给每个图标设置：

```swift
shape.strokeEnd = 0.001
shape.lineCap = .Round
```

如果我们用图标的中心点定位，结束和开始状态下的菜单看起来会是下图这样：

![](http://www.c4ios.com/images/cosmos/18/03.png)

![](http://www.c4ios.com/images/cosmos/18/04.png)

如果我们使用第一个点作为形状的 `anchorPoint`，我们会得到下面这样的效果：

![](http://www.c4ios.com/images/cosmos/18/05.png)

![](http://www.c4ios.com/images/cosmos/18/06.png)

我们的想法是，在结束状态下使用 `anchorPoint`，另外一个状态使用 `center`。不过，如果我们不停的转换位置，实现起来会很复杂。所以我们必须使用 `anchorPoint` 计算实际的 `center`，使用这个 `center` 在不同的状态下来换转换。

开始行动吧。

## 2. 获取图标

第一步就是从符号库里获取图标，更新图标的锚点。

打开 `MenuIcons.swift` 文件，在类里添加下列代码：

```swift
func taurus() -> Shape {
    let shape = AstrologicalSignProvider.sharedInstance.taurus().shape
  shape.anchorPoint = Point()
  return shape
}

func aries() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.aries().shape
  shape.anchorPoint = Point(0.0777,0.536)
  return shape
}

func gemini() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.gemini().shape
  shape.anchorPoint = Point(0.996,0.0)
  return shape
}

func cancer() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.cancer().shape
  shape.anchorPoint = Point(0.0,0.275)
  return shape
}

func leo() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.leo().shape
  shape.anchorPoint = Point(0.379,0.636)
  return shape
}

func virgo() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.virgo().shape
  shape.anchorPoint = Point(0.750,0.387)
  return shape
}

func libra() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.libra().shape
  shape.anchorPoint = Point(1.00,0.559)
  return shape
}

func pisces() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.pisces().shape
  shape.anchorPoint = Point(0.099,0.004)
  return shape
}

func aquarius() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.aquarius().shape
  shape.anchorPoint = Point(0.0,0.263)
  return shape
}

func sagittarius() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.sagittarius().shape
  shape.anchorPoint = Point(1.0,0.349)
  return shape
}

func capricorn() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.capricorn().shape
  shape.anchorPoint = Point(0.288,0.663)
  return shape
}

func scorpio() -> Shape {
  let shape = AstrologicalSignProvider.sharedInstance.scorpio().shape
  shape.anchorPoint = Point(0.255,0.775)
  return shape
} 
```

每个方法都从生成器里获取了图标，设置锚点为图标的开始点。我们不需要知道符号的其他信息（比如：大和小、点和线等等），所以我们只需返回修改过锚点的形状。

接下来，我们还要存储符号的副本，用于之后的各种操作。创建一个字典变量来存储符号和方法，这些方法会给符号创建不同的样式：

```swift
var signIcons : [String:Shape]!
```

接着，把下列方法添加到类里：

```swift
func createSignIcons() {
    signIcons = [String:Shape]()
    signIcons["aries"] = aries()
    signIcons["taurus"] = taurus()
    signIcons["gemini"] = gemini()
    signIcons["cancer"] = cancer()
    signIcons["leo"] = leo()
    signIcons["virgo"] = virgo()
    signIcons["libra"] = libra()
    signIcons["scorpio"] = scorpio()
    signIcons["sagittarius"] = sagittarius()
    signIcons["capricorn"] = capricorn()
    signIcons["aquarius"] = aquarius()
    signIcons["pisces"] = pisces()
    
    for shape in [Shape](self.signIcons.values) {
        shape.strokeEnd = 0.001 // 合并下面两步所做的内容
        shape.lineCap = .Round  // strokeEnd 设置成 0.001，做一个很小的点
        shape.lineJoin = .Round // 形状的开始部分
        
        shape.transform = Transform.makeScale(0.64, 0.64, 1.0)
        shape.lineWidth = 2
        shape.strokeColor = white
        shape.fillColor = clear
    }
}
```

我们从库里拿出来的符号还是原始符号，需要缩小一点。如果我们不进行这行操作：`hape.transform = Transform.makeScale(0.64, 0.64, 1.0)`，符号的形状就会超级大，如下图：

![](http://www.c4ios.com/images/cosmos/18/07.png)

## 3. 目标位置

接下来，我们需要计算每个形状的目标位置，存储到两个数组中。在类里添加下面的代码：

```swift
var innerTargets : [Point]!
var outerTargets : [Point]!
```

接着添加下列方法：

```swift
func positionSignIcons() {
    innerTargets = [Point]()
    let provider = AstrologicalSignProvider.sharedInstance
    let r = 10.5
    let dx = canvas.center.x
    let dy = canvas.center.y
    for i in 0..<provider.order.count {
        let ϴ = M_PI/6 * Double(i)
        let name = provider.order[i]
        if let sign = signIcons[name] {
            sign.center = Point(r * cos(ϴ) + dx, r * sin(ϴ) + dy)
            canvas.add(sign)
            sign.anchorPoint = Point(0.5,0.5)
            innerTargets.append(sign.center)
        }
    }
    
    outerTargets = [Point]()
    for i in 0..<provider.order.count {
        let r = 129.0
        let ϴ = M_PI/6 * Double(i) + M_PI/12.0
        outerTargets.append(Point(r * cos(ϴ) + dx, r * sin(ϴ) + dy))
    }
}
```

上述过程使用了以下概念：

1. 创建形状，通过 `anchorPoint` 方法来给每个形状设置起始点
2. 调用 `shape.center` 实际上返回的是形状的位置 `anchorPoint`，相对于每个形状的 `superview` 
3. 调整 `anchorPoint` 到形状的中心（例如 `{0.5,0.5}`），不会改变形状的位置
4. 重置 `anchorPoint` 后，我们能够获取形状实际的中心位置，作为目标
 
最后，在 `createSignIcons` 的结尾，给所有的符号都设置了样式后，添加下列代码：

```swift
positionSignIcons()
```

现在，让我们把这些东西都整合起来。

### 3.1 检查一下

将菜单的 `backgroundColor` 值改成 `C4Purple`，创建符号图标，`setup()` 方法应该像下面这样：

```swift
public override func setup() {
    canvas.backgroundColor = COSMOSbkgd
    createSignIcons()
}
```

到 `WorkSpace` 里，更新 `setup()` 方法，如下：

```swift
override func setup() {
    canvas.add(MenuIcons().canvas)
}
```

运行程序，效果如下：

![](http://www.c4ios.com/images/cosmos/18/08.png)

## 4. 图标的动画效果

有四个不同的动画需要创建：位置的出和入，形状的出现和隐藏。所以，创建四个动画变量：

```swift
var signIconsOut : ViewAnimation!
var signIconsIn : ViewAnimation!
var revealSignIcons : ViewAnimation!
var hideSignIcons : ViewAnimation!
```

增加下列方法：

```swift
func createSignIconAnimations() {
}
```
需要四个动画是由于设计原因，的吧是线在出入时不同的方向有不同的顺序。

出：移动，出现；入：隐藏，移动。

由于模式正好相反，当动画开始后，我们需要获取单独的动画来应对移动和隐藏，形成序列。在方法里添加如下内容：

```swift
revealSignIcons = ViewAnimation(duration: 0.5) {
    for sign in [Shape](self.signIcons.values) {
        sign.strokeEnd = 1.0
    }
}
revealSignIcons?.curve = .EaseOut

hideSignIcons = ViewAnimation(duration: 0.5) {
    for sign in [Shape](self.signIcons.values) {
        sign.strokeEnd = 0.001
    }
}
hideSignIcons?.curve = .EaseOut
```

需要标注一下：
 
1. 将每个动画分开，更容易在每一步进行自定义
2. 能够明确每个方向的持续时间
3. 把 stroke 设置回 `0.001` 用于显示小点

现在，把下列代码添加到 `createSignIconAnimations`：

```swift
signIconsOut = ViewAnimation(duration: 0.33) {
    for i in 0..<AstrologicalSignProvider.sharedInstance.order.count {
        let name = AstrologicalSignProvider.sharedInstance.order[i]
        if let sign = self.signIcons[name] {
            sign.center = self.outerTargets[i]
        }
    }
}
signIconsOut?.curve = .EaseOut

// 把图标移动到结束位置
signIconsIn = ViewAnimation(duration: 0.33) {
    for i in 0..<AstrologicalSignProvider.sharedInstance.order.count {
        let name = AstrologicalSignProvider.sharedInstance.order[i]
        if let sign = self.signIcons[name] {
            sign.center = self.innerTargets[i]
        }
    }
}
signIconsIn?.curve = .EaseOut
```

我们在这里获取了图标，设置图标在内环时的中心点和在外环时的中心点。

### 4.1 查看一下效果

调用出现和移动动画来测试一下，如下：

```swift
func animOut() {
    delay(1.0) {
        self.signIconsOut?.animate()
    }

    delay(1.5) {
        self.revealSignIcons?.animate()
    }
    
    delay(2.5) {
        self.animIn()
    }
}

func animIn() {
    delay(0.25) {
        self.hideSignIcons?.animate()
    }

    delay(1.0) {
        self.signIconsIn?.animate()
    }

    delay(2.5) {
        self.animOut()
    }
}
```

在 `setup()` 里调用 `animOut()`，如下： 

```swift
public override func setup() {
    canvas.backgroundColor = COSMOSbkgd
    createSignIcons()
    createSignIconAnimations()
    animOut()
}
```

效果如下：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/NeighboringTerribleFieldspaniel.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 5. 清理

删除 `animIn` 和 `animOut` 方法，修改 `setup()` 方法如下：

```swift
public override func setup() {
    canvas.frame = Rect(0,0,80,80)
    canvas.backgroundColor = clear
    createSignIcons()
    createSignIconAnimations()
}
```

获取 [MenuIcons.swift](https://gist.github.com/C4Framework/131a89c7f7cd0286eca4) 文件。

干净利索！