---
title: Xib加载的几种方法
permalink: xibjia-zai-de-ji-chong-fang-fa
id: 33
updated: '2016-08-25 12:50:56'
date: 2016-08-13 15:00:33
tags:
---



> 一，本质

 xib本质是`XML`代码（在编译时`Xcode`将`xib`中内容转换成代码）

注：如果一个`view`是从`xib`中加载出来的，默认`width`与`height`是`xib`中描述的尺寸，frame中(x,y)值默认为零

> 二、控制器中加载xib

加载方式一：

```
NSArray *newsArr = [[NSBundle mainBundle] loadNibNamed:@"news" owner:nil options:nil];
UIView *newsView = newsArr.firstObject;
```

注：
“`loadNibNamed:owner:options`” 方法返回值是一个`NSArray`，因为一个xib中可以放多个`view`，但一般情况我们都只放一个在`xib`中

加载方式二：

```
UINib *nib = [UINib nibWithNibName:@"news" bundle:[NSBundle mainBundle]];//[NSBundle mainBundle]作为参数时，可以传nil，切记[NSBundle mainBundle]调用其他方法时不可以为nil，用nil调用方法等于什么操作都没做
UIView *news = [[nib instantiateWithOwner:nil options:nil] firstObject];
```

> 三、使用xib加载view的注意事项

1，如果一个`view`是从`xib`加载出来的，那么`xib`绑定的`View`初始化过程中，不会执行`init`方法和`initWithFrame`方法，因此在页面中如果通过 `alloc init` 来初始化该`view`，界面会是空白
2，如果多个页面中用到该`view`，最好在`xib`绑定的类中提供快速创建的类方法

```
+ (instancetype)viewForXib
{
    return [[[NSBundle mainBundle] loadNibNamed:NSStringFromClass(self) owner:nil options:nil] firstObject];
}
```