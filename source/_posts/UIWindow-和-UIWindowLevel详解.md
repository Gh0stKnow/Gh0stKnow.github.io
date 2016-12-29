---
title: UIWindow 和 UIWindowLevel详解
tags: 'iOS'
permalink: uiwindow-he-uiwindowlevelxiang-jie
id: 10
updated: '2016-06-23 22:41:08'
date: 2016-06-11 10:13:09
---



一、UIWindow是一种特殊的UIView，通常在一个程序中只会有一个UIWindow，但可以手动创建多个UIWindow，同时加到程序里面。UIWindow在程序中主要起到三个作用：

　　1、作为容器，包含app所要显示的所有视图

　　2、传递触摸消息到程序中view和其他对象

　　3、与UIViewController协同工作，方便完成设备方向旋转的支持

二、通常我们可以采取两种方法将view添加到UIWindow中：

　　1、addSubview

　　直接将view通过addSubview方式添加到window中，程序负责维护view的生命周期以及刷新，但是并不会为去理会view对应的ViewController，因此采用这种方法将view添加到window以后，我们还要保持view对应的ViewController的有效性，不能过早释放。

　　2、rootViewController

　　rootViewController时UIWindow的一个遍历方法，通过设置该属性为要添加view对应的ViewController，UIWindow将会自动将其view添加到当前window中，同时负责ViewController和view的生命周期的维护，防止其过早释放

三、WindowLevel

　　UIWindow在显示的时候会根据UIWindowLevel进行排序的，即Level高的将排在所有Level比他低的层级的前面。下面我们来看UIWindowLevel的定义：

```
　　　　const UIWindowLevel UIWindowLevelNormal;
　　　　const UIWindowLevel UIWindowLevelAlert;
　　　　const UIWindowLevel UIWindowLevelStatusBar;
　　　　typedef CGFloat UIWindowLevel;
```

　　[iOS](http://lib.csdn.net/base/swift)系统中定义了三个window层级，其中每一个层级又可以分好多子层级(从UIWindow的头文件中可以看到成员变量CGFloat _windowSublevel;)，不过系统并没有把则个属性开出来。UIWindow的默认级别是UIWindowLevelNormal，我们打印输出这三个level的值分别如下：



1. 2012-03-27 22:46:08.752 UIViewSample[395:f803] Normal window level: 0.000000  
2. 2012-03-27 22:46:08.754 UIViewSample[395:f803] Normal window level: 2000.000000  
3. 2012-03-27 22:46:08.755 UIViewSample[395:f803] Normal window level: 1000.000000  

　　这样印证了他们级别的高低顺序从小到大为Normal < StatusBar < Alert，下面请看小的测试代码：

　　运行结果如下图：

![img](http://pic002.cnblogs.com/images/2012/302680/2012032722535669.jpg)

　　我们可以注意到两点：

　　1）我们生成的normalWindow虽然是在第一个默认的window之后调用makeKeyAndVisible，但是仍然没有显示出来。这说明当Level层级相同的时候，只有第一个设置为KeyWindow的显示出来，后面同级的再设置KeyWindow也不会显示。

　　2）statusLevelWindow在alertLevelWindow之后调用makeKeyAndVisible，淡仍然只是显示在alertLevelWindow的下方。这说明UIWindow在显示的时候是不管KeyWindow是谁，都是Level优先的，即Level最高的始终显示在最前面。

