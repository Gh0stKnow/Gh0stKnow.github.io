---
title: willMoveToSuperView导致观察者无法释放bug
tags: 'bug归档'
permalink: willmovetosuperviewdao-zhi-guan-cha-zhe-wu-fa-shi-fang-bug
id: 35
updated: '2016-09-10 10:39:20'
date: 2016-09-10 10:33:17
---



![obeserve无法释放](http://ww1.sinaimg.cn/large/801b780agw1f7oaf0t13bj20rt0hp44z.jpg)


(void)willMoveToSuperview 默认不做任何事情；当接收者父视图将要改变的时候回来到该方法，其中newSuperview是将要被添加的视图，该参数可以为new，子类可以重写这方法来作为特定的实现。
最近在开发公司项目的过程中，遇到了一个比较棘手的bug。
同事在构造一个modal出来的Controller中，用到了某个三方库
其中有用到KVO去监听某个属性的值。
而且也在dealloc里面对观察者进行了销毁。

但在点击关闭按钮销毁该界面的时候，程序先回到父控制器窗口，然后crash。
查了许久原因未果。

通过断点调试和查看函数调用栈发现
observeValueForKeyPath:这个方法一共来了两次
进一步调试发现
是- (void)willMoveToSuperview:(UIView *)newSuperview这个方法调用了两次，而增加观察者方法就写在其中。
两次调用的时间分别是View视图将要显示前，和将要销毁前。

查阅资料得知newSuperview可以为nil，也就是modal视图销毁时这个方法会进行第二次调用，即把视图添加到为nill的View上。

进而在方法内部做了对参数的校验，解决问题。

不过把观察者写在view里面始终是野路子，把逻辑操作都写在controller中才是正途。

