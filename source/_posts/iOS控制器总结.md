---
title: iOS控制器总结
tags: 'iOS'
permalink: ioskong-zhi-qi-tiao-zhuan-zong-jie
id: 3
updated: '2016-06-28 00:44:44'
date: 2016-06-07 11:32:50
---




![banner.jpg](https://ooo.0o0.ooo/2016/06/08/5757fb07bad17.jpg)

以FirstViewController(FVC)的按钮button点击后跳转到SecondViewController(SVC)为例说明:

# 方式一：Storyboard的segues方式
鼠标点击按钮button然后按住control键拖拽到SVC页面，在弹出的segue页面中选择跳转模式即可



segues方式
优点:操作方便,无代码生成,在storyboard中展示逻辑清晰

缺点:页面较多时不方便查看,团队合作时可维护性差, 多人合作时不建议使用这种方式。

# 方式二：选项卡UITabBarController控制器
通过调用UITabBarController的addChildViewController方法添加子控制器，代码实例如下：

UITabBarController *tabbarVC = [[ UITabBarController alloc ] init ];

FirstViewController *FVC = [[FirstViewController ] init ];

FVC.tabBarItem.title = @"控制器1";

FVC.tabBarItem.image = [ UIImage imageNamed : @"first.png"];

SecondViewController *SVC = [[SecondViewController ] init ];

SVC.tabBarItem.title = @"控制器2";

SVC. tabBarItem.image = [UIImage imageNamed : @"new.png"];

// 添加子控制器（这些子控制器会自动添加到UITabBarController的 viewControllers 数组中）

[tabbarVC addChildViewController :FVC];

[tabbarVC addChildViewController :SVC];


优点:代码量较少

缺点:tabbar的ios原生样式不太好看,(不常用,目前不建议使用),如果要使用,建议自定义tabbar

# 方式三：导航控制器UINavigationController
// 在FVC的button的监听方法中调用:

[self.navigationController pushViewController:newC animated:YES];//跳转到下一页面

// 在SVC的方法中调用:

[self.navigationController popViewControllerAnimated:YES];//返回上一页面

//当有多次跳转发生并希望返回根控制器时,调用:

[self .navigationController popToRootViewControllerAnimated: YES ];//返回根控制器,即最开始的页面


# 方式四：利用 Modal 形式展示控制器


// 在FVC中调用:

[ self presentViewController:SVC animated: YES completion:nil];

// 在SVC中调用:

[ self dismissViewControllerAnimated: YES completion: nil ];

# 方式五：直接更改 UIWindow 的 rootViewController




总结：
Storyboard方式适合个人开发小程序时使用,有团队合作或者项目较大时不建议使用

UITabBarController因为目前系统的原生样式不太美观,不建议使用

推荐使用UINavigationController和Modal,无明显缺点,而且目前大部分程序都使用这两种方式,只是看是否需要导航控制器而确定使用哪种方案

