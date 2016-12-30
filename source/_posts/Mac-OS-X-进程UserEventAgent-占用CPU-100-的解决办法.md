---
title: Mac OS X 进程UserEventAgent 占用CPU 100%的解决办法
permalink: mac-os-x-jin-cheng-usereventagent-zhan-yong-cpu-100-de-jie-jue-ban-fa
id: 20
updated: '2016-06-26 22:54:49'
date: 2016-06-26 22:53:59
tags:
---



前几天电脑无故发热特别厉害，达到了70+度，一查进程居然有一个UserEventAgent占了100%的CPU，虽未影响系统流畅度，还是受不了这么高的温度，我也不记得对系统和硬件做过什么大的改动，上网查了一下似乎和USB设备有关，后来一忙又把这事放了几天，今天干脆直接申请了Apple支持，电话连线近一小时仍无果（服务还是不错的），只能提供了一些后续的解决方案。 
闲不住自己试了几遍，发现只有连接鼠标的时候才会有这个问题（不过这鼠标以前用着一切正常..），点开UserEventAgent进程发现打开的文件里很多在/system/Library/下，没看到和鼠标有关的，直接Finder定位到/System/Library/搜索mouse，出来两个目测和鼠标有关的文件： 
AppleHIDMouse.kext 
AppleHIDMouseAgent.plugin 

本想直接把这文件移动到其他路径，为求保险再搜了一下上面自己猜测的信息，在Apple的技术支持论坛找到一个和上述一样解决方法的帖子，原来只需要删除下面这一个文件即可： 
```/System/Library/UserEventPlugins/AppleHIDMouseAgent.plugin/Contents/MacOS/AppleH IDMouseAgent 
```

删除之，重启，终于恢复正常。