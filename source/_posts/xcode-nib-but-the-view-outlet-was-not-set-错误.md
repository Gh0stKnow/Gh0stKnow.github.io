---
title: xcode nib but the view outlet was not set 错误
permalink: xcode-nib-but-the-view-outlet-was-not-set-cuo-wu
id: 11
updated: '2016-07-13 15:01:20'
date: 2016-06-11 22:48:50
tags:
---





xib 中, 没有对File's Owner 的Outlets view 进行绑定, 导致在父视图中插入子视图时出错, 在IB中拖拽Files' Owner到view, 添加绑定后, 运行成功!
总结一下创建视图和绑定的步骤:
1. 创建控制器. File->New File->Iphone OS->Cocoa Touch Class->UIViewController subclass;
2. 创建xib. File->New File->Iphone OS->User Interface->View XIB
3. 绑定controller和view. 用Interface Builder打开xxx.xib, 点击Files' Owner, 在Identity Inspector里面的Class Identity, 选择Step 1创建的控制器类, 接着拖拽File's Owner到View中, 选择Outlets->view.先选中file's owner(这个很重要)