# Chapter 15 圆形菜单

在应用的中心放一个菜单，Jake 这个想法非常漂亮，不过实现起来也非常难。特别是还要计算动画时间，实现流畅的动画效果。首先需要分解这个动画，分解成一个一个的视觉组件；然后分解每个组件的动画效果，尤其是打开关闭时的动画效果；接着，我们处理一下手势、targets、actions，打磨一下 UI 里的 label 显示效果。

![](http://www.c4ios.com/images/cosmos/15/01.png)

## 1. 思路

思路非常简单：屏幕中心有一个居中的图标，点击后转换成一个圆形的菜单，里面有 12 个星座图标按钮，点击某个星座按钮后，菜单又回到一开始的样子：屏幕中心的一个图标按钮。非常明了。

![菜单开始时的状态](http://www.c4ios.com/images/cosmos/15/02.png)

为了实现这样的效果，我们会分别处理每个组件的动画，然后把各个组件组合在一起，形成最后的效果。

![](http://www.c4ios.com/images/cosmos/15/03.png)

### 1.1. 圆环

菜单里的主要视觉效果就是出现的一系列的圆环，在快要结束的时候，会出现两个圆环和一些点，打开状态下实际上有七个圆环：一个粗一点的，五个细一点的，一个带有破折号的。最上面这个有十一个分割线，把十二个图标分割开来。在这一部分我们会实现基本的动画，然后实现剩下的交互功能，最终实现完整的菜单。

![](http://www.c4ios.com/images/cosmos/15/04.png)

### 1.2. 图标

图标是整个应用的核心部分，每个图标有不同的位置。这些图标不仅仅是看着好看，动画效果也让整个菜单看起来栩栩如生：在关闭状态下，这些图标看起来像是一些点，在打开状态时，会展现完整的样子、从关闭到打开的过程，我们需要一个非常棒的移动过程，通过动画展现整个过程。

![](http://www.c4ios.com/images/cosmos/15/05.png)

### 1.3. 选择器

菜单出现后，当用户点击某个图标按钮，我们希望该按钮的背景高亮，表示这个按钮被选中。有两个方法可以实现：我们可以生成十二个形状并更改它们的填充色；我们还可以创建一个遮罩形状并旋转它。无论是哪种方法，我们都需要思考用户使用哪种手势来触发点击。

![](http://www.c4ios.com/images/cosmos/15/06.png)

### 1.4. 手势

正确的手势会对我们的应用产生很大影响。我们希望应用里的交互简单明了，同时想让用户看到一些新奇的手势功能。我们还要想办法解决打开状态、选择图标和关闭状态之间的平衡，在这个过程中不能有太多的手势和动作。

### 1.5. UI

为了给菜单和应用增加更多内容，我们要增加一个标题 label，显示当前选中的图标；一个 info 按钮，能够打开信息面板；一个说明 label，仅出现在用户第一次打开应用，还没点击菜单时（比如，如果用户不知道如何使用菜单，该 label 显示使用说明）。最后，我们还要增加一点阴影效果，在打开菜单时阴影消失。有了阴影的对比，在弹出菜单时，菜单和背景就可以区分开。

### 1.6. 动画

所有的状态都要遵循下列动画效果：

1. 所有的动画都是从中心向外扩展，看起来像是从一条粗线里出来的。
2. 分割线的动画效果是随机的。
3. 图标开始是一个点，然后慢慢向外扩大直到到达最后的位置，从点扩大成最终的外形。
4. 转换最后出现 info 按钮
5. 在转换过程中，阴影渐渐消失。
6. 选中一个图标后，显示高亮效果。
7. 用户关闭菜单，或者选择 info 按钮后，所有的动画反向进行。

## 2. 分解

把所有的工作分解成下列步骤，每个步骤单独开发，最后将所有的步骤组合在一起：

1. 创建 MenuRings 类，处理菜单的基本外形
2. 创建 MenuIcons 类，处理图标的呈现和动画效果
3. 创建 MenuSelector 类，处理手势、选择器、信息页按钮和菜单里的标签
4. 创建 MenuShadow 类，处理阴影的动画效果
5. 创建 Menu 类，把上述的效果组合在一起