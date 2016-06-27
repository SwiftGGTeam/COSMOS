# Chapter 21 完成菜单功能

现在我们要开始完成 `Menu.swift` 中的代码，计划是：

1. 把所有的图层都添加到这个类的画布中
2. 给这个类的画布添加一个手势识别器
3. 添加一个方法来隐藏或者显示菜单
4. 在用户打开或关闭菜单时，播放声音
5. 添加一些提示标签来帮助用户使用

## 1. 添加所有的层

创建下面的类变量：

```swift
var menuRings : MenuRings!
var menuIcons : MenuIcons!
var menuSelector : MenuSelector!
var menuShadow : MenuShadow!
var shouldRevert = false
```

然后，修改 `setup()` 方法为：

```swift
override func setup() {
    // 设置透明背景 
    canvas.backgroundColor = clear
    // 让画布尽可能小
    canvas.frame = Rect(0,0,80,80)

    // 创建圆环
    menuRings = MenuRings()
    
    // 创建菜单选择按钮
    menuSelector = MenuSelector()
    
    // 创建图标
    menuIcons = MenuIcons()

    // 创建阴影
    menuShadow = MenuShadow()
    menuShadow.canvas.center = canvas.bounds.center
    
    // 添加各个部件
    canvas.add(menuShadow.canvas)
    canvas.add(menuRings.canvas)
    canvas.add(menuSelector.canvas)
    canvas.add(menuIcons.canvas)
}
```

然后，在工程的 `WorkSpace` 中修改 `setup()` 为：

```swift
override func setup() {
    canvas.backgroundColor = COSMOSbkgd
    let menu = Menu()
    menu.canvas.center = canvas.center
    canvas.add(menu.canvas)
}

```

如果现在运行 app，你会看见：

![](http://c4ios.com/cosmos/01.png)

可以看到圆环和图标都显示出来了。

## 2. 添加手势

现在，回到 `MenuSelector.swift`, 剪切整个 `createGesture()` 并复制到这个类里。

你会看到 Xcode 有一个报错：

![](http://c4ios.com/cosmos/02.png)

要修复很简单，只需要把 

```swift
self.
```

改为

```swift
self.menuSelector.
```

最终，你的方法会像这样：

```swift
func createGesture() {
    canvas.addLongPressGestureRecognizer { (locations, center, state) -> () in
        switch state {
        case .Changed:
            self.menuSelector.update(center)
        case .Cancelled, .Ended, .Failed:
            self.menuSelector.currentSelection = -1
            self.menuSelector.highlight.hidden = true
            self.menuSelector.menuLabel.hidden = true
        default:
            _ = ""
        }
    }
}
```

现在，手势识别器已经是菜单层的一部分，而不再属于菜单选择器层。

## 3. 显示菜单

现在，我们希望在菜单打开或关闭的时候伴随着一系列有序的动画。为了实现这功能，添加以下两个方法到你的类里：

```swift
func revealMenu() {
    menuShadow.reveal?.animate()
    menuRings.thickRingOut?.animate()
    menuRings.thinRingsOut?.animate()
    menuIcons.signIconsOut?.animate()
    
    delay(0.33) {
        self.menuRings.revealHideDividingLines(1.0)
        self.menuIcons.revealSignIcons?.animate()
    }
    delay(0.66) {
        self.menuRings.revealDashedRings?.animate()
        self.menuSelector.revealInfoButton?.animate()
    }
}

func hideMenu() {
    menuRings.hideDashedRings?.animate()
    menuSelector.hideInfoButton?.animate()
    menuRings.revealHideDividingLines(0.0)
    
    delay(0.16) {
        self.menuIcons.hideSignIcons?.animate()
    }
    delay(0.57) {
        self.menuRings.thinRingsIn?.animate()
    }
    delay(0.66) {
        self.menuIcons.signIconsIn?.animate()
        self.menuRings.thickRingIn?.animate()
        self.menuShadow.hide?.animate()
        self.canvas.interactionEnabled = true
    }
}
```

### 3.1. 菜单是否可见

大多数交互都是需要菜单处于可见的状态，所以我们会创建一个检查菜单是否可见的变量，在你的类中添加如下代码：

```swift
var menuIsVisible = false

```

在 `revealMenu()` 方面的最后添加：

```swift
delay(1.0) {
   self.menuIsVisible = true
}
```

我们知道在显示菜单的时候，动画需要花 1 秒钟时间，所以我们设置变量也需要遵循这个逻辑。

在 `hideMenu()` 方法的头部添加：

```swift
self.menuIsVisible = false
```

这里没有延迟，因为菜单一旦触发隐藏，那菜单就是不可见状态。

你需要把同样的逻辑添加到 `revealMenu()` 方法中，从而保证状态的正确性。

你的显示和隐藏方法最终会变成这样：

```swift
func revealMenu() {
    menuIsVisible = false
    menuShadow.reveal?.animate()
    menuRings.thickRingOut?.animate()
    menuRings.thinRingsOut?.animate()
    menuIcons.signIconsOut?.animate()
    
    delay(0.33) {
        self.menuRings.revealHideDividingLines(1.0)
        self.menuIcons.revealSignIcons?.animate()
    }
    delay(0.66) {
        self.menuRings.revealDashedRings?.animate()
        self.menuSelector.revealInfoButton?.animate()
    }
    delay(1.0) {
        self.menuIsVisible = true
    }
}

func hideMenu() {
    self.menuIsVisible = false
    menuRings.hideDashedRings?.animate()
    menuSelector.hideInfoButton?.animate()
    menuRings.revealHideDividingLines(0.0)
    
    delay(0.16) {
        self.menuIcons.hideSignIcons?.animate()
    }
    delay(0.57) {
        self.menuRings.thinRingsIn?.animate()
    }
    delay(0.66) {
        self.menuIcons.signIconsIn?.animate()
        self.menuRings.thickRingIn?.animate()
        self.menuShadow.hide?.animate()
        self.canvas.interactionEnabled = true
    }
}
```

## 4. 行为的逻辑实现

我们首先需要为手势的处理添加一些逻辑。在 switch 语句的 `.Cancelled` 分支中第一个 `if` 后面添加如下代码：

```swift
if self.menuIsVisible {
    self.hideMenu()
} else {
    self.shouldRevert = true
}
```

如果手势结束、被取消或者失败，这个逻辑会检查两件事：如果菜单是可见的则需要反转，否则就设置 `shouldRevert` 让菜单在打开完毕后自动关闭。

然后，在 `revealMenu()` 方法中的 `delay(1.0)` 函数块，添加：

```swift
if self.shouldRevert {
    self.hideMenu()
    self.shouldRevert = false
}
```

一旦菜单显示完毕，就会检查它是否需要反转。

然后在 switch 语句的开头添加：

```swift
case .Began:
   self.revealMenu()

```

当手势开始时（默认按 0.25 秒之后），则开启菜单。

最后，在 `hideMenu()` 方法的 `delay(0.66)` 块中添加：

```swift
self.canvas.interactionEnabled = true

```

以及添加如下的代码到手势逻辑中的 `.Cancelled` 分支：

```swift
self.canvas.interactionEnabled = f

```
这可以有效的防止用户在菜单正在返回关闭状态时对其进行操作。同理，在菜单关闭完成后，需要重新设置它为 true。

你的 `createGesture()` 方法应该像这样：


```swift
func createGesture() {
    // 添加长按的手势
    canvas.addLongPressGestureRecognizer { (locations, center, state) -> () in
        switch state {
        case .Began:
            self.revealMenu()
        case .Changed:
            self.menuSelector.update(center)
        case .Cancelled, .Ended, .Failed:
            self.menuSelector.currentSelection = -1
            self.menuSelector.highlight.hidden = true
            self.menuSelector.menuLabel.hidden = true
            if self.menuIsVisible {
                self.hideMenu()
            } else {
                self.shouldRevert = true
            }
            self.canvas.interactionEnabled = false
        default:
            _ = ""
        }
    }
}
```

现在，千万要确定你在 `Menu` 类的 setup 方法中调用了它。

![](https://zippy.gfycat.com/EnragedLimpDungenesscrab.mp4)

酷！

## 5. 声音

给菜单添加打开和关闭的声音。添加如下两个类变量：

```swift
let hideMenuSound = AudioPlayer("menuClose.mp3")!
let revealMenuSound = AudioPlayer("menuOpen.mp3")!

```

在 `setup()` 方法中，调整其参数：

```swift
hideMenuSound.volume = 0.64
revealMenuSound.volume = 0.64

```

然后，在 `revealMenu()` 方法的顶部，添加：

```swift
revealMenuSound.play()

```

同样，在 `hideMenu()` 的顶部，也添加：

```swift
hideMenuSound.play()

```

运行一遍听听看，很有趣的声音。


## 6. 隐藏的提示

让我们添加一个提示标签来告诉用户长按菜单这个操作

> 我们发现在用户明白如何使用这个 app 之前，需要被提示才能知道去长按菜单。所以我们添加这个标签来帮助他们。我们还需要假设用户在阅读一次提示之后就不再需要看第二次，所以我们还需要防止提示重复出现。

创建如下的类变量：

```swift
var instructionLabel : UILabel!
```

> 记住我们使用 `UILabel` 因为我们希望有两行居中显示的文字， `TextShape` 并不满足这个要求。

添加如下的方法来创建提示：


```swift
func createInstructionLabel() {
    instructionLabel = UILabel(frame: CGRect(x: 0,y: 0,width: 320, height: 44))
    instructionLabel.text = "press and hold to open menu\nthen drag to choose a sign"
    instructionLabel.font = UIFont(name: "Menlo-Regular", size: 13)
    instructionLabel.textAlignment = .Center
    instructionLabel.textColor = .whiteColor()
    instructionLabel.userInteractionEnabled = false
    instructionLabel.center = CGPointMake(view.center.x,view.center.y - 128)
    instructionLabel.numberOfLines = 2
    instructionLabel.alpha = 0.0
    canvas.add(instructionLabel)
}
```

然后创建一个计时器来控制标签的显示：

```swift
var timer : Timer!

```

然后，创建一下两个方法来显示和隐藏提示：


```swift
func showInstruction() {
    ViewAnimation(duration: 2.5) {
        self.instructionLabel?.alpha = 1.0
        }.animate()
}

func hideInstruction() {
    ViewAnimation(duration: 0.25) {
        self.instructionLabel?.alpha = 0.0
        }.animate()
}
```

注意到我们在 `show()` 方法里停止了计时器。之所以这么做是因为我们希望计时器只被触发一次。你可能会想：“那为什么不用过延时触发显示？”。因为我们希望在提示现实之前用户就打开了菜单的情况下，停止这个计时器。如果用 delay 来实现的话，不能被中途取消。

下一步，创建一个计时器并在 `setup()` 函数的末尾调用 `createInstructionLabel()`：

```swift
createInstructionLabel()

timer = Timer(interval: 5.0) {
    self.showInstruction()
}
timer?.start()
```

上述代码设置了在主画布初始化完毕 5 秒后显示菜单，这个时间无论对知道如何通过长按菜单按钮来使用 app 的用户还是初次使用的用户都足够友好。

现在，在 `revealMenu()` 方法的头部添加 `hideInstruction()` 的调用， 这里有一个小逻辑，就是只有在提示标签可见的时候，才会执行这一段代码：


```swift
if instructionLabel?.alpha > 0.0 {
    hideInstruction()
}
```

同时，在 `revealMenu()` 方法的顶部添加：

```swift
timer.stop()
```swift


运行一下：

![](https://zippy.gfycat.com/ConventionalDistantHartebeest.mp4)

酷毙了。


## 7. 阴影

很简单。

在 `revealMenu()` 方法的顶部添加：

```swift
menuShadow.reveal?.animate()
```

然后，在 `hideMenu()` 方法的 `delay(0.66)` 中添加：

```swift
self.menuShadow.hide?.animate()


```

现在，在菜单打开的时候，背景会变暗。


## 8. 结束

至此，整个菜单的工作已经完成了 98%。下一章讲会把所有东西放到一起，包括为菜单添加交互的事件，信息页面，以及视差效果的背景。

这里是 [Menu.swift](https://gist.github.com/C4Framework/2235949aec0fddd3b8da) 的一份拷贝。


喝杯啤酒，休息一下。
