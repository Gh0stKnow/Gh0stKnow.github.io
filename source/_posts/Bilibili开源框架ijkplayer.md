---
title: Bilibili开源框架ijkplayer
permalink: bilibilikai-yuan-kuang-jia-ijkplayer
id: 18
updated: '2016-06-28 13:11:04'
date: 2016-06-26 09:47:59
tags:
---



ijkplayer 是一款做视频直播的框架, 基于ffmpeg, 支持 Android 和 iOS, 网上也有很多集成说明, 但是个人觉得还是不够详细, 在这里详细的讲一下在 iOS 中如何集成ijkplayer, 即便以前从没有接触过, 按着下面做也可以集成成功!

# 一. 下载ijkplayer

ijkplayer下载地址:[https://github.com/Bilibili/ijkplayer](https://github.com/Bilibili/ijkplayer)

ijkplayer下载地址:[https://github.com/Bilibili/ijkplayer](https://github.com/Bilibili/ijkplayer)
下载完成后解压


# 二. 编译 ijkplayer

说是编译 ijkplayer, 其实是编译 ffmpeg, 在这里我们已经下载好了`ijkplayer`, 所以 github 上`README.md`中的`Build iOS`那一步中有一些步骤是不需要的.

说是编译 ijkplayer, 其实是编译 ffmpeg, 在这里我们已经下载好了`ijkplayer`, 所以 github 上`README.md`中的`Build iOS`那一步中有一些步骤是不需要的.
下面开始一步一步编译:

说是编译 ijkplayer, 其实是编译 ffmpeg, 在这里我们已经下载好了`ijkplayer`, 所以 github 上`README.md`中的`Build iOS`那一步中有一些步骤是不需要的.
下面开始一步一步编译:
1.打开终端, cd 到`jkplayer-master`文件夹中, 也就是下载完解压后的文件夹

2.执行命令行`./init-ios.sh`, 这一步是去下载 ffmpeg 的, 时间会久一点, 耐心等一下.如下图:

![img](http://upload-images.jianshu.io/upload_images/1803339-96d2217e323aab4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.在第2步中下载完成后, 执行`cd ios`, 也就是进入到 ios目录中

4.进入 ios 文件夹后, 在终端依次执行`./compile-ffmpeg.sh clean`和`./compile-ffmpeg.sh all`命令, 编译 ffmpeg, 也就是`README.md`中这两步
*编译时间较久, 耐心等待一下.*

# 三. 打包`IJKMediaFramework.framework`框架

集成 ijkplayer 有两种方法:

集成 ijkplayer 有两种方法:
一种方法是按照`IJKMediaDemo`工程中那样, 直接导入工程`IJKMediaPlayer.xcodeproj`, 在这里不做介绍

第二种集成方法是把 ijkplayer 打包成`framework`导入工程中使用. 下面开始介绍如何打包`IJKMediaFramework.framework`

1. 首先打开工程`IJKMediaPlayer.xcodeproj`, 位置如下图:

![img](http://upload-images.jianshu.io/upload_images/1803339-607cc84c212faf90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.工程打开后设置工程的 scheme

3.设置好 scheme 后, 分别选择真机和模拟器进行编译, 编译完成后, 进入 Finder, 如下图:

![img](http://upload-images.jianshu.io/upload_images/1803339-344cda905745ff39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进入 Finder 后, 可以看到有真机和模拟器两个版本的编译结果

下面开始合并真机和模拟器版本的 framework, 注意不要合并错了
打开终端, 进行合并, 命令行具体格式为:

```
lipo -create "真机版本路径" "模拟器版本路径" -output "合并后的文件路径"
```

下面很重要, 需要用合并后的`IJKMediaFramework`把原来的`IJKMediaFramework`替换掉
上图中的1、2两步完成后, 绿色框住的那个`IJKMediaFramework.framework`文件就是我们需要的框架了, 可以复制出来, 稍后我们需要导入工程使用.

# 四. iOS工程中集成ijkplayer

新建工程, 导入合并后的`IJKMediaFramework.framework`以及相关依赖框架以及相关依赖框架
导入框架后, 在`ViewController.m`进行测试, 首先导入`IJKMediaFramework.h`头文件, 编译看有没有错, 如果没有错说明集成成功.

导入框架后, 在`ViewController.m`进行测试, 首先导入`IJKMediaFramework.h`头文件, 编译看有没有错, 如果没有错说明集成成功.
接着开始在`ViewController.m`文件中使用`IJKMediaFramework`框架进行测试使用, 写一个简单的直播视频进行测试, 在这里看一下运行后的结果, 后面会放上 Demo 供下载.

