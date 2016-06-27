# Chapter 17 圆环菜单的动画效果

尽管我把这一章从上章拆分出来了，但是我们还是要在 `MenuRings.swift` 文件里进行开发。运行应用，确保你看到的应用效果和下图一致：

![](http://www.c4ios.com/images/cosmos/17/01.png)

> 如果不是这个效果，请查看上一章节，确保所有的变量设置正确，特别是菜单里的每条线、每个内环的变量值。

## 1. 粗圆环的动画效果

我们会对每个组件设置进出效果和转换效果。对于粗圆环的动画效果来说，我们先在类里添加下面两个变量：

```swift
var thickRingOut : ViewAnimation?
var thickRingIn : ViewAnimation?
```

现在，创建一个方法，写代码：

```swift
func createThickRingAnimations() {
    thickRingOut = ViewAnimation(duration: 0.5) {
        self.thickRing?.frame = self.thickRingFrames[1]
    }
    thickRingOut?.curve = .EaseOut
    
    thickRingIn = ViewAnimation(duration: 0.5) {
        self.thickRing?.frame = self.thickRingFrames[0]
    }
    thickRingIn?.curve = .EaseOut
}
```

这个方法设置了两个动画，第一个 out 设置的是粗圆环的新 frame，告诉 shape 更新自己的路径，强制 shape 重画路径直到符合 frame。第二个则与 out 相反。

让我们测试一下。

创建下面这两个方法：

```swift
func animOut() {
    delay(1.0) {
        self.thickRingOut?.animate()
    }
    delay(2.0) {
        self.animIn()
    }
}

func animIn() {
    delay(1.0) {
        self.thickRingIn?.animate()
    }
    delay(2.0) {
        self.animOut()
    }
}
```

接着在 setup 里输入下列代码：

```swift
self.createThickRingAnimations()
animOut()
```

得到了这样的效果：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/OldWindyKiwi.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 2. 细圆环的动画效果（淡出）

先给我们的细圆环创建动画效果，和之前的步骤有些不同，这次我们需要操作五条线，所以我们使用循环体来创建并存储到数组中。

在类里增加下面两个变量：

```swift
var thinRingsOut : ViewAnimationSequence?
```

接着，添加下面的方法：

```swift
func createThinRingsOutAnimations() {
    var animationArray = [ViewAnimation]()
    for i in 0..<self.thinRings.count-1 {
        let anim = ViewAnimation(duration: 0.075 + Double(i) * 0.01) {
            let circle = self.thinRings[i]
            // 除了第一个之外其他都会动
            if (i > 0) {
                // 把圆环的不透明度调整到 1.0
                ViewAnimation(duration: 0.0375) {
                    circle.opacity = 1.0
                    }.animate()
            }
            
            circle.frame = self.thinRingFrames[i+1]
        }
        anim.curve = .EaseOut
        animationArray.append(anim)
    }
    thinRingsOut = ViewAnimationSequence(animations: animationArray)
}
```

这个方法会创建一系列动画变量，基于很多章节之前设置的目标 frame 的值来填充动画。

这个方法创建了一个动画数组，然后运行循环：

1. 获取每个细圆环，从小到大
2. 创建动画，持续时间基于当前的 index`(0.075 + Double(i) + 0.01)`
3. 检查每一行的 index 是否大于零，从而判断是不是内环
4. 对于任何一个不是内环的圆环，创建一个不透明的动画，执行完毕后立即消失
5. 获取当前目标的 frame，也就是当前 index + 1
6. 设置当前圆环的 frame，调用 `updatePath`
7. 设置动画的曲线为 `.EaseOut`
8. 把动画添加到数组里
 
循环结束后，就构建好了 `thinRingsOut` 序列。

## 3. 细圆环动画效果（进入）

进入的动画和移出的动画非常相似，仅仅有几处地方不同。

在类里增加下列变量：

```swift
var thinRingsIn : ViewAnimationSequence?
```

增加下面的方法：

```swift
func createThinRingsInAnimations() {
    var animationArray = [ViewAnimation]()
    for i in 1...self.thinRings.count {
        let anim = ViewAnimation(duration: 0.075 + Double(i) * 0.01, animations: { () -> Void in
            let circle = self.thinRings[self.thinRings.count - i]
            if self.thinRings.count - i > 0 {
                ViewAnimation(duration: 0.0375) {
                    circle.opacity = 0.0
                    }.animate()
            }
            circle.frame = self.thinRingFrames[self.thinRings.count - i]
        })
        anim.curve = .EaseOut
        animationArray.append(anim)
    }
    thinRingsIn = ViewAnimationSequence(animations: animationArray)
}
```

不同之处在于：

1. 这里的数学计算表示循环体和出的效果正好相反：`self.thinRings.count - 1`
2. 透明度调整为 `0.0`
3. 由于我们总是想获取数组“之前”的数字（比如 `-1`），所以我们的循环执行如下：`for i in 1...self.thinRings.count`

## 4. 虚线环的动画效果

要完成这些动画，还需要创建两个新的动画变量：

```swift
var revealDashedRings : ViewAnimation?
var hideDashedRings : ViewAnimation?
```

增加下列方法：

```swift
func createDashedRingAnimations() {
    revealDashedRings = ViewAnimation(duration: 0.25) {
        self.dashedRings[0].lineWidth = 4
        self.dashedRings[1].lineWidth = 12
    }
    revealDashedRings?.curve = .EaseOut
    
    hideDashedRings = ViewAnimation(duration: 0.25) {
        self.dashedRings[0].lineWidth = 0
        self.dashedRings[1].lineWidth = 0
    }
    hideDashedRings?.curve = .EaseOut
}
```

### 4.1. 查看一下效果

想要看出、入效果，需要修改 `animOut` 和 `animIn` 方法，如下：

```swift
func animOut() {
    delay(1.0) {
        self.thickRingOut?.animate()
        self.thinRingsOut?.animate()
        self.revealDashedRings?.animate()
    }
    delay(2.0) {
        self.animIn()
    }
}

func animIn() {
    delay(1.0) {
        self.thickRingIn?.animate()
        self.thinRingsIn?.animate()
        self.hideDashedRings?.animate()
    }
    delay(2.0) {
        self.animOut()
    }
}
```

在 setup() 里调用 animOut()：

```swift
createThinRingsOutAnimations()
createThinRingsInAnimations()
createDashedRingAnimations()
```

运行，效果如下图：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/OldNastyJaguarundi.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

估计现在你已经注意到了，圆环和竖线的调速有些问题。不用担心，最后把所有的组件组合到一起后，会解决这个问题的。

快要到达终点了。

## 5. 分割线

分割线的动画效果比较简单，不过 Jake 要求它们出现的顺序随机。翻译过来就是：每次线出现的动画和顺序都是要随机的，要和之前的有所不同。

我是这样解决这个问题的：

```swift
func revealHideDividingLines(target: Double) {
    var indices = [0,1,2,3,4,5,6,7,8,9,10,11]
    
    for i in 0...11 {
        delay(0.05*Double(i)) {
            let randomIndex = random(below: indices.count)
            let index = indices[randomIndex]
            
            ViewAnimation(duration: 0.1) {
                self.menuDividingLines[index].strokeEnd = target
            }.animate()
            
            indices.removeAtIndex(randomIndex)
        }
    }
}
```

把这些添加到类里。

这个方法做了下面这些事情：

1. 获取一个目标（应该是 `0.0` 或者 `1.0`）
2. 创建 index 数组
3. 运行循环，执行 12 次
4. 每次遍历都从下标数组中随机获取一个元素
5. 然后，把分割线的 `strokeEnd` 移动到对应随机下标的 `target` 值
6. 执行完毕之后从下标数组中删除这个元素

接着，测试一下效果：

```swift
func animOut() {
    delay(1.0) {
        self.thickRingOut?.animate()
        self.thinRingsOut?.animate()
    }

    delay(1.5) {
        self.revealHideDividingLines(1.0)
    }

    delay(2.5) {
        self.animIn()
    }

}

func animIn() {
    delay(0.25) {
        self.revealHideDividingLines(0.0)
    }

    delay(1.0) {
        self.thickRingIn?.animate()
        self.thinRingsIn?.animate()
    }
    delay(2.0) {
        self.animOut()
    }
}
```

效果如下：

{% raw %}
  <video id="my-video" class="video-js" controls preload="auto" width="1000" height="400"
  poster="MY_VIDEO_POSTER.jpg" data-setup="{}">
  <source src="https://zippy.gfycat.com/IdenticalLazyElectriceel.mp4" type='video/mp4'>
  <p class="vjs-no-js">
    To view this video please enable JavaScript, and consider upgrading to a web browser that
    <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a>
  </p>
  </video>
{% endraw %}

## 6. 清理代码

现在，不要调用 `self.animOut()` 方法并删掉两个动画方法。

接着创建两个新方法：

```swift
func createRingsLines() {
    createThickRing()
    createThinRings()
    createDashedRings()
    createMenuDividingLines()
}

func createRingsLinesAnimations() {
    createThickRingAnimations()
    createThinRingsOutAnimations()
    createThinRingsInAnimations()
    createDashedRingAnimations()
}
```

更新 `setup` 方法如下：

```swift
public override func setup() {
  canvas.backgroundColor = clear
  canvas.frame = Rect(0,0,80,80)
  createRingsLines()
  createRingsLinesAnimations()
}
```

最后，在类里再添加一个变量：

```swift
var menuIsVisible = false
```

我们在之后的章节中会用到这最后一个变量。

[MenuRings.swift](https://gist.github.com/C4Framework/9d44e5800ed86446df4a) 源文件。