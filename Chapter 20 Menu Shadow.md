# Chapter 20 菜单阴影

和上一章马拉松式的编程相比，这一章就是小菜一碟。 我们需要在菜单下方显示阴影，也就是完成下面三件事情：

1. 一个阴影对象
2. 展示动画
3. 隐藏动画

和之前我们为不同的层创建不同的 `CanvasController` 对象保持一致，这次我们也需要这么做。

在 `MenuShadow.swift` 添加如下的代码：

```swift
var reveal : ViewAnimation?
var hide : ViewAnimation?

public override func setup() {
    canvas.frame = Rect(UIScreen.mainScreen().bounds)
    canvas.backgroundColor = black
    canvas.opacity = 0.0
    createShadowAnimations()
}

func createShadowAnimations() {
    reveal = ViewAnimation(duration:0.25) {
        self.canvas.opacity = 0.44
    }
    reveal?.curve = .EaseOut
    
    hide = ViewAnimation(duration:0.25) {
        self.canvas.opacity = 0.0
    }
    hide?.curve = .EaseOut
}

```

这就 OK 了。

这个类很简单，这里就不测试它了。我们在下一章集成菜单的全部功能的时候会看到它的效果。

这里是 [MenuShadow.swift](https://gist.github.com/C4Framework/afdda9aacef565588e26) 一份拷贝。

## 1. 曙光

认真的说，我们已经看到了黎明的曙光。

你可以完成这个教程。

我相信你<sup><a name="to1" href="#1">1</a></sup>
。

---

#### 脚注

<a name="1">1.</a> 我也相信我自己……这里有很多写作工作！<a href="#to1">↩</a>