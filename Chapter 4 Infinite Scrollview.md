# Chapter 4 无限滚动视图

有时候你需要一个普通的滚动视图，还有些时候你可能想让视图一直滚啊滚。所谓的无限滚动视图，就是当内容滚动到最后一项内容时，再从第一个视图开始继续滚动，无限循环。

在这一章里，你会学到如何重写 `UIScrollView` 里的 `layoutSubviews()` 方法，判断在需要的时候更新 scrollview 的位置。

视图的内容的大小应该包含 12「屏」。

![](http://www.c4ios.com/images/cosmos/4/01.gif)

## 1. 溢出边界

无限滚动视图的关键点在于，不是到头或尾的时候发生弹跳效果，而是将内容循环显示。

要做到这一点，你需要知道：

1. 一共有多少内容（例如视图的 `contentSize`）
2. 内容可以分成多少屏，这个需要你自己计算
3. 特定位置时，用户将视图滚动到的偏移位置（你需要通过视图的 `currentOffset` 得知）

![在示例里，内容占据了了 12 屏](http://www.c4ios.com/images/cosmos/4/02.png)

### 1.1. 规则

你需要遵守两个规则：
 
1. 当用户滑动超过了内容的结尾时，需要做什么
2. 当用户滑动超过了内容的开头时，需要做什么
 
#### 1.1.1 滚动视图的行为

`UIScrollView` 的行为非常简单：不管什么时候用户滚动，view 就会更新变量 `contentOffset`，然后判断是否需要更新显示的内容。

### 1.2. 小于开始部分时

需要不断检查内容偏移是否小于视图原点的最左边：

```swift
if contentOffset.x < 0 {
	// 让内容移动到结束位置 
}
```

![在这个例子里，contentOffset.x 小于 0（视图的原点）](http://www.c4ios.com/images/cosmos/4/03.png)

#### 1.2.1. 大于结束部分时

需要检查内容偏移是否大于整个视图结束的位置：

```swift
if contentOffset.x > totalWidth - view.width {
	// 让内容移动到开始位置
}
```

这一步和之前的方法有些不同，因为 `contentOffset` 指的是视图的原来的起点。官方对 `contentOffset` 的解释是：

Content view 的基准点相对于 Scrollview 为原点的偏移值。

所以，当内容滚动到超过终点的位置就是判断原点的偏移量（校者注：此时的 origin 的位置即为 contentOffset.x）加上单个视图的宽度（view.width）是否超过了总长度（totalWidth，自行计算）。

![在这里，contentOffset + width 越过了结束位置](http://www.c4ios.com/images/cosmos/4/04.png)

### 1.3. 创建它吧

已经准备好可以创建无限滚动视图了。正如你所见，并不需要做太多工作，只需要几行代码而已。神奇的事情将在下一章发生。