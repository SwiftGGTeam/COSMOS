# Chapter 24 提交到应用市场

最后一个任务是将 COSMOS 提交到应用市场。在上传前有一些事情要做，而且一旦上了 iTunes Connect 还有其他的事情要处理。

## 1. 优化

第一步是优化应用，在 Xcode 中，点击 

`CMD+I`

…或者选择：`Product > Profile`

![](http://www.c4ios.com/images/cosmos/24/01.png)

这样就打开了 Instruments。
选择 Allocations 工具然后运行 App。

![](http://www.c4ios.com/images/cosmos/24/02.png)

App 启动了，我随便操作了一会，接着停止检测，看看内存都用在哪、增长了多少等等…… 

![](http://www.c4ios.com/images/cosmos/24/03.png)

我还是简单解释一下比较好：

1. 我看到总的内存用量非常多，不过只有 23.5 MB 是持久性内存用量，这些内存用量都在 All Heap & Anonymous VM 里。

2. 进入 All Heap & Anonymous VM，把这些用量按照自身大小排序。

3. 排在第一位的内存用量是 16 MB，不过看起来像是系统自带的东西，所以我暂时先不管它。

4. 接下来第二位的内存用量看起来像是图片、音频文件，然后从这之后内存用量的大小就变得非常小了（到了第十一行，都小于 100 kb）。 

5. 我知道我放了一个很大的启动图片，还有很多小的星星图片以及装饰图片，等等。所以，31 MB 比较正常，无需担心，可以继续往下操作了。

接着运行 **Leaks**，结果没有发生什么有趣的事情。

我分析了一下 App，Xcode 没有给我警告，也没有提示有潜在的问题……

![](http://www.c4ios.com/images/cosmos/24/04.png)

所以，我现在十分确定，我可以把应用上传到应用市场上了。

## 2. iTunes Connect

我用我的开发者帐号登录 iTunes Connect，选择 **My Apps**。

![](http://www.c4ios.com/images/cosmos/24/05.png)

接下来创建新的 App……<sup><a name="to1" href="#1">1</a></sup>


![](http://www.c4ios.com/images/cosmos/24/06.png)

开始填写所需的信息……

![](http://www.c4ios.com/images/cosmos/24/07.png)

……不过当我填写到 **Bundle ID** 部分时，我想起来我还没有给 App 设置相关代码呢。

## 3. App ID & Profile

所以我登录我的开发者帐号，创建一个新的 **App ID**……

![](http://www.c4ios.com/images/cosmos/24/08.png)

……它的 identifier 是 **com.c4ios.COSMOS**。

![](http://www.c4ios.com/images/cosmos/24/09.png)

接着我创建了新的 provisioning profile，用来上传到应用市场……

![](http://www.c4ios.com/images/cosmos/24/10.png)

这个 App ID 也就是刚刚我创建的那个 App……

![](http://www.c4ios.com/images/cosmos/24/11.png)

## 4. Xcode

回到 Xcode，打开菜单栏上的 prferences，点击 Accounts。双击我的账户，看到 Xcode 已经看到了我刚刚在网上创建的新的配置文件了。

![](http://www.c4ios.com/images/cosmos/24/12.png)

点击配置文件旁边的 **download** 按钮，下载安装，然后找到 **Build Settings**，在 **Code Signing** 部分，把配置文件改成 **COSMOS**…… 

![](http://www.c4ios.com/images/cosmos/24/13.png)

接着，我把 code signing identity 改成我的主账号……

![](http://www.c4ios.com/images/cosmos/24/14.png)

然后，我尝试 **Archive** 该工程，不过我收到了错误警告，说我的 **Bundle Identifier** 是错误的。

所以，到工程的 **.plist** 文件中把 bundle identifier 改成我在 provisioning profile 时使用的那个。

![我把 $(PRODUCT_BUNDLE_IDENTIFIER) 改成了 com.c4ios.COSMOS](http://www.c4ios.com/images/cosmos/24/15.png)

……到工程的 **Build Settings** 中的 **Packaging** 完成同样的操作……

![将 C4.COSMOS 改成 com.c4ios.COSMOS](http://www.c4ios.com/images/cosmos/24/17.png)

再次 archive 工程，又收到了另外一个错误警告，说：

> Entitlements file do not match those specified in your provisioning profile.(0xE8008016)

我到 Stack Overflow 上搜了一下如何解决这个问题……

[关于 Entitlements Issue 的答案](http://stackoverflow.com/questions/22625785/entitlements-file-do-not-match-those-specified-in-your-provisioning-profile-0xe)

看完答案后我回到 Xcode，更新工程里的 Capabilities 部分中的 Game Center 和 In-app Purchases……

![](http://www.c4ios.com/images/cosmos/24/18.png)

这样看起来好像解决问题了，我模糊记得这两个部分的 App ID 默认值就是我创建的（我不能不选它们）。

![](http://www.c4ios.com/images/cosmos/24/19.png)

再次 archive……

![](http://www.c4ios.com/images/cosmos/24/20.png)

这次成功了，不过，当我上传到应用市场时，我又收到了一个错误提示：

![](http://www.c4ios.com/images/cosmos/24/21.png)

我把设置中的 version 改成 **1.2**……<sup><a name="to2" href="#2">2</a></sup>

![](http://www.c4ios.com/images/cosmos/24/22.png)

archive 然后再次上传，看起来好像成功了！<sup><a name="to3" href="#3">3</a></sup>

![](http://www.c4ios.com/images/cosmos/24/23.png)

接着，上传完成，我还是收到了错误警告……

![](http://www.c4ios.com/images/cosmos/24/24.png)

我觉得，这个警告的意思是，发现这个 App 使用了自定义的 C4 API，需要有人来审核一下我的 App，因为到目前位置，机器还没有比人类更聪明。

## 5. 第二次的 iTunes Connect

我回到 iTunes Connect，检查我刚刚上传的 App 的状态。

![](http://www.c4ios.com/images/cosmos/24/25.png)

上面说 App 目前处于 “Processing” 的状态，我不明白这是什么意思……不过我知道，我得等待一段时间了。

所以，我开始写出详细步骤……<sup><a name="to4" href="#4">4</a></sup>

当我写完这部分时，我看到状态已经不是 processing 了。

![](http://www.c4ios.com/images/cosmos/24/26.png)

我选择 new build，然后更新 Test 信息。

![](http://www.c4ios.com/images/cosmos/24/27.png)

删除旧版本的 COSMOS，从 **App Store** 的 tab 页上选择 new build。

![](http://www.c4ios.com/images/cosmos/24/28.png)

![](http://www.c4ios.com/images/cosmos/24/29.png)

然后把版本号改成 1.2，提交申请。

出现一些确认信息的对话框，然后 App 上传了，等待人工审核。

![](http://www.c4ios.com/images/cosmos/24/30.png)

这是星期天的下午……

## 6. 发布

COSMOS 在我提交申请到几天后通过了审核，我们需要时间来建设我们的网站，在 11 月 23 日，我们上线了。

---

#### 脚注

<a name="1">1.</a> 这篇文章并不是现在写的，因为我在六月份的时候上传过一个版本。不过，我想添加这一步以供你参考，之前的应用是用旧版本的 C4 写的，所以我需要彻底重写代码……现在已经十月了。<a href="#to1">↩</a>

<a name="2">2.</a> 我实际上做的是 1.1，不过我还是得到了同样的错误警告，因为这里已经有 1.1 了，就是六月份发布的那个版本，我忘了这个版本了。<a href="#to2">↩</a>

<a name="3">3.</a> 一旦你看到上传的进度条，就意味着你终于设置对了相关信息，App 可以上传了。<a href="#to3">↩</a>

<a name="4">4.</a> 我截屏了，所以在上传的这段时间，我可以用来写这篇文章。<a href="#to4">↩</a>