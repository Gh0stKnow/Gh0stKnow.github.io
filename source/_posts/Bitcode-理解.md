---
title: Bitcode 理解
permalink: bitcode-li-jie
id: 19
updated: '2016-06-28 13:10:38'
date: 2016-06-26 15:14:20
tags:
---



今天试着用Xcode 7 beta 3在真机(iOS 8.3)上运行一下我们的工程，结果发现工程编译不过。看了下问题，得到的信息是我们引入的一个第三方库不包含bitcode。嗯，不知道bitcode是啥，所以就得先看看这货是啥了。

![](http://cc.cocimg.com/api/uploads/20150702/1435823658708967.png)
# Bitcode是什么？

找东西嘛，最先想到的当然是先看官方文档了。在[App Distribution Guide – App Thinning (iOS, watchOS)](https://developer.apple.com/library/prerelease/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AppThinning/AppThinning.html#//apple_ref/doc/uid/TP40012582-CH35)一节中，找到了下面这样一个定义：

Bitcode is an intermediate representation of a compiled program. Apps you upload to iTunes Connect that contain bitcode will be compiled and linked on the App Store. Including bitcode will allow Apple to re-optimize your app binary in the future without the need to submit a new version of your app to the store.

说的是bitcode是被编译程序的一种中间形式的代码。包含bitcode配置的程序将会在App store上被编译和链接。bitcode允许苹果在后期重新优化我们程序的二进制文件，而不需要我们重新提交一个新的版本到App store上。

嗯，看着挺高级的啊。

继续看，在[What’s New in Xcode-New Features in Xcode 7](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/WhatsNewXcode/Articles/xcode_7_0.html)中，还有一段如下的描述

Bitcode. When you archive for submission to the App Store, Xcode will compile your app into an intermediate representation. The App Store will then compile the bitcode down into the 64 or 32 bit executables as necessary.

当我们提交程序到App store上时，Xcode会将程序编译为一个中间表现形式(bitcode)。然后App store会再将这个botcode编译为可执行的64位或32位程序。

再看看这两段描述都是放在App Thinning(App瘦身)一节中，可以看出其与包的优化有关了。喵大(@onevcat)在其博客[开发者所需要知道的 iOS 9 SDK 新特性](http://onevcat.com/2015/06/ios9-sdk/)中也描述了iOS 9中苹果在App瘦身中所做的一些改进，大家可以转场到那去研读一下。

# Bitcode配置

在上面的错误提示中，提到了如何处理我们遇到的问题：

You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target. for architecture arm64

要么让第三方库支持，要么关闭target的bitcode选项。

实际上在Xcode 7中，我们新建一个iOS程序时，bitcode选项默认是设置为YES的。我们可以在”Build Settings”->”Enable Bitcode”选项中看到这个设置。

不过，我们现在需要考虑的是三个平台：iOS，Mac OS，watchOS。

- 对应iOS，bitcode是可选的。
- 对于watchOS，bitcode是必须的。
- Mac OS不支持bitcode。

如果我们开启了bitcode，在提交包时，下面这个界面也会有个bitcode选项：

![blob.png](http://cc.cocimg.com/api/uploads/20150817/1439791780516322.png)

盗图，我的应用没办法在这个界面显示bitcode，因为依赖于第三方的库，而这个库不支持bitcode，暂时只能设置ENABLE_BITCODE为NO。

所以，如果我们的工程需要支持bitcode，则必要要求所有的引入的第三方库都支持bitcode。我就只能等着公司那些大哥大姐们啥时候提供一个新包给我们了。

## 题外话

如上面所说，bitcode是一种中间代码。LLVM官方文档有介绍这种文件的格式，有兴趣的可以移步[LLVM Bitcode File Format](http://llvm.org/docs/BitCodeFormat.html#llvm-bitcode-file-format)。

### 参考

- [App Distribution Guide – App Thinning (iOS, watchOS)](https://developer.apple.com/library/prerelease/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AppThinning/AppThinning.html#//apple_ref/doc/uid/TP40012582-CH35)
- [What’s New in Xcode-New Features in Xcode 7](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/WhatsNewXcode/Articles/xcode_7_0.html)
- [开发者所需要知道的 iOS 9 SDK 新特性](http://onevcat.com/2015/06/ios9-sdk/)
- [LLVM Bitcode File Format](http://llvm.org/docs/BitCodeFormat.html#llvm-bitcode-file-format)

