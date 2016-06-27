# Chapter 0 准备开始

我已经为你建好了一个基本上是空的工程项目，通过教程一起来从头学习 COSMOS。这是一个 Xcode 工程，它包含需要你添加代码的空的类、所有的资源文件（包括音频）、以及一个本地版本的 C4。

[COSMOS Xcode Project](http://www.c4ios.com/cosmos/CosmosEmpty.zip)

让我们先来看看工程里的东西是如何组织的。

现在打开工程。

## 1. 工程导航

当你在 Xcode 打开项目的时候，你会看到如下图窗口左半部分的东西：

![工程导航](http://www.c4ios.com/images/cosmos/0/navigator.png)

这个是整个工程的导航结构，你可以自己组织文件目录、查找、定位所有你项目中的文件。

## 2. Swift 文件

这里一共有 7 个 swift 文件：

* AppDelegate
* ViewController
* Menu
* ParallaxBackground
* AstrologicalSignProvider
* InfiniteScrollview
* InfoPanel

在这些文件中，你将不会接触 `AppDelegate` 文件，但是你会频繁地与其他文件打交道。

在整个教程中，每个章节将会详细指出哪个文件是你需要添加代码的。举个例子，在「无限滚动视图」章节中，你的代码都会添加到 `InfiniteScrollview.swift` 文件中。

## 3. 资源

有两种类型的资源：图片和音频。所有的音频文件都在一个文件夹内并且都是 `mp3` 文件。所有图片资源都包含在一个叫 Assets.xcassets 的组里面，以图片集（image set）的方式组织，实际上其中每个「图像」都是由 `2x` 和 `3x` 两个版本构成。

## 4. C4

在这个工程中，我已经打包了所有 C4 文件的静态副本。这个教程并**不是**用来教会你如何使用 C4，但对于本教程组织文件的方式，可以确保我们今天编写的代码对应写这个教程时存在的 C4 中文件目录。

> 我们嵌入 C4 工程的方法与其他形式的嵌入 C4 到 Xcode 工程的方法没有什么本质区别。二者最终都是一样的效果，除了后者的代码可能可以与任何提交到 GitHub 主仓库上最新的版本保持一致。

在整篇教程中的任何一处，你都不需要访问 C4 文件的源代码。但是你如果对底层发生的事情有进一步挖掘的兴趣，也可以随时查看代码，因为，它们就在那里。