# Chapter 12 小星星

除了要考虑小星星的速度以及对应的不同图片，添加小星星的方式和添加背景区别不大。

打开 `StarsSmall.swift` 并添加如下的代码到 init 方法中：

```swift
var signOrder = AstrologicalSignProvider.sharedInstance.order
contentSize = CGSizeMake(frame.size.width * (1.0 + CGFloat(signOrder.count) * gapBetweenSigns), 1.0)
signOrder.append(signOrder[0])
```

接着，创建一个 `for` 循环来添加所有的小星星，这些小星星的位置是从 provider 里获取的。代码如下：

```swift
// 将所有的小星星添加到视图中
for i in 0..<signOrder.count {
    // 计算偏移量
    let dx = Double(i) * Double(frame.size.width * speed * gapBetweenSigns)
    // 创建一个变换
    let t = Transform.makeTranslation(Vector(x: Double(center.x) + dx, y: Double(center.y), z: 0))
    // 获取当前的符号
    if let sign = AstrologicalSignProvider.sharedInstance.get(signOrder[i]) {
        // 每个点创建一个小星星
        for point in sign.small {
            let img = Image("6smallStar")!
            var p = point
            p.transform(t)
            img.center = p
            add(img)
        }
    }
}
```

简单吧。

`SmallStars.swift` 完整的代码可以在[这里](https://gist.github.com/C4Framework/6eae5a284153f8c7c88e)下载。

## 1. 测试之

通过工程中 `WorkSpace` 以下的代码来检查小星星的方向：

```swift
canvas.backgroundColor = COSMOSbkgd
let smallStars = StarsSmall(frame: view.frame, speed: 1.0)
canvas.add(smallStars)
```

![](http://c4ios.com/images/cosmos/12/01.png)

背景需要深色的，因为图片本身是白色的，不然的话就看不清楚了。