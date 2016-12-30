---
title: 升级Ruby配置CocoaPods安装
permalink: jie-jue-cocoapodan-zhuang-rubyban-ben-di-de-wen-ti
id: 31
updated: '2016-08-07 13:25:25'
date: 2016-08-07 13:05:29
tags:
---



这两天全新安装了系统，安装cocoapods时出现：

> activesupport requires Ruby version >= 2.2.2

检查下发现淘宝源的ruby版本是2.0.0，于是使用RVM更新了下Ruby。



## 1、安装 RVM

RVM:Ruby Version Manager,Ruby版本管理器，包括Ruby的版本管理和Gem库管理(gemset)

> curl -L get.rvm.io | bash -s stable 

等待一段时间后就可以成功安装好 RVM。

> source ~/.bashrc  
>
> source ~/.bash_profile  

测试是否安装正常

> rvm -v 

![13:03:18.jpg](http://upload-images.jianshu.io/upload_images/1727086-68742171c589c6ad.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2、用RVM升级Ruby
```
查看当前ruby版本  
ruby -v  
列出已知的ruby版本  
rvm list known  

<!------MORE------->


安装ruby 2.2.2  
rvm install 2.2.2  
```


安装完成之后ruby -v查看是否安装成功。
终端输出
```

kidteaing-macmini:~ GeXiaodong$ ruby -v
ruby 2.2.2p95 (2015-04-13 revision 50295) [x86_64-darwin14]
```

到此已经完成ruby的升级

