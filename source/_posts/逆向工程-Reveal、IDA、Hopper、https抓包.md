---
title: 逆向工程 - Reveal、IDA、Hopper、https抓包
tags: [iOS, 转载]
permalink: ni-xiang-gong-cheng-reveal-ida-hopper-httpszhua-bao
id: 16
updated: '2016-07-13 15:00:10'
date: 2016-06-23 11:21:14
---



*iOS应用的安全性 常常被大家忽视*

##**一、iOS 如何做才安全:**

**详见《iOS如何做才安全》**

 

##**二、ipa文件**

1、AppStore里的ipa包 可以通过 iTunes 下载到电脑。iOS8.3以下系统的非越狱的手机上，可以用MAC上的PP助手等软件，直接把手机上的ipa文件(包含沙盒里的存储文件)拷贝到电脑。

如果是越狱手机，都可以用PP助手、itools直接把ipa导出到电脑，并且可以用PP助手、iExplorer、itools这些工具 查看 iOS的系统目录。

 

MAC上安装 iExplorer软件，用iExplorer 可以看到 手机（非越狱也可以） 在 iTunes上备份的内容。

如果你在帮测试美女的手机 调试问题的时候， 在 iTunes上设置 “连接次iPhone时自动同步”(或者点击 备份到本地电脑),默认该手机上的照片、短信等内容都会备份到你的电脑上，用 iExplorer 就可以看到 这位 美女的隐私。

曾经有次不小心看到同事的隐私信息，所以现在都比较注意这块，避免引发误会。

 

2、拿到ipa文件后，解压缩，得到.app文件，右键显示包内容，可以看到里面的app中的图片、js、plist、静态H5页 等资源。

比如 你要 用微信里的默认表情包，解压微信的ipa包就可以获取到。

 

3、iOS的系统目录和MAC上的都类似（类unix系统）。iOS系统的目录图：

 ![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160603113527508-253137717.png)

 <!------MORE------->

##**三、沙盒 中的数据**

iPhone上 计算器的沙盒：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602201509805-146936001.png)

.app文件:应用程序本身的数据，打包时候的一些资源文件（如：图片、plist等文件）、可执行文件。这个目录不会被iTunes同步。

Documents ：存储不可再生 的关键数据。不会被iTunes同步

Library：保存配置文件和其他一些文件。NSUserDefault 会存储到 Library下的Preferences中 的 plist文件中。可以直接打开，所以 也不要在 NSUserDefault 中存一些 关键数据，或者 存储的时候 进行 AES等方式的加密。

Library/Caches可以用来保存可再生的数据，比如网络请求，用户需要负责删除对应文件。 

这个目录（除了Library/Caches外）会被iTunes同步

tmp：临时文件。不需要的时候，手动将其内文件删除。（当应用不再运行的时候，系统可能会将此目录清空。） 

tmp：临时文件。不需要的时候，手动将其内文件删除。（当应用不再运行的时候，系统可能会将此目录清空。） 
这个目录不会被iTunes同步

存到沙盒的数据都是不安全的，关键数据一定 要做加密存储。

 

##**四、Reveal 工具：查看 任何APP 的UI结构**

1、不越狱的手机 可以用 Reveal 来查看自己APP的UI结构。不能查看其他APP的UI结构。这里就不再描述了。

 

 2、越狱手机 上可以查看 任何APP的UI结构。

在越狱的手机上，在 Cydia 搜索并安装 Reveal Loader，如果搜索不到。就 点下面的“软件源”，选择“BigBoss”,选择“全部软件包”，点右边R的字母，去一个个找到 Reveal Loader，放心吧，你一定能找到的，我用的iOS7.1的系统测试的，没问题。

安装完成后，打开“设置”页面，下拉到最底部，点击“Reveal”

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602192309977-1694804093.png)

 

3、点击 Enabled Applictions 。然后选中 你想分析的APP。

4、确保iOS和OSX在同一个IP网段内。打开想分析的 APP，如果该APP已经启动，则关闭后再次启动

5、打开MAC上的 Reveal，选中 左上方列表里的 APP，**比如QQ：**

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602193213742-452768022.png)

 

6、如果 Reveal 左上方 一直显示：No Connection。说明iPhone上的 Reveal Loader 没安装成功，需要配置一下。

首先从MAC上 用PP助手或 iTools 查看“文件系统（系统）”--》Library文件夹，看 Library文件夹下面有没有 RHRevealLoader 文件夹，如果没有，就 右键 新建文件夹，并修改名字为：RHRevealLoader。

在Mac 下打开已经安装的Reveal，选择标题栏**Help**下的**Show Reveal Library in Finder  下的 iOS library  **选项，将会显示如下界面：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602194122961-7619153.png)

将**libReveal.dylib **文件通过**PP助手**拷贝到刚才创建的`RHRevealLoader`文件夹下，就可以了。

 

然后 从手机上打开APP， 再 打开 MAC上的 Reveal 软件，左上方 就会出来 相关APP的选项。

再发个 **淘宝中的天猫模块**吧：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602195031367-1190080065.png) 

 

##**五、反编译工具：IDA**

从AppStore下载的ipa都是加壳的(苹果 把开发者上传的ipa进行了加壳再放到AppStore中)，加壳的ipa要先去壳，可以用clutch、dumpdecrypted、使用gdb调试 等解密去壳工具，这个我们后面再说。

如果你有越狱手机，可以直接 从 PP助手上下载ipa包，这个就是 脱壳后的。。

现在 我们先反编译 自己的APP，通过Xcode打包的APP 都是没加壳的，可以直接用来反编译。

 

新建一个项目，在 ViewController 的 viewDidLoad 方法里 加 几句代码。

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(50, 70, 200, 100)];
    label.text = @"CeShiLabel007";
    label.backgroundColor = [UIColor redColor];
    [self.view addSubview:label];
}
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

将项目 打包，生成 ipa文件，下面我们就用IDA分析一下 ipa。

将ipa文件 解压后 得到.app文件：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602200635492-1438112164.png)

 

下载IDA，并打开：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602200012914-225461018.png)

点击“New”按钮，选择刚才 解压的 .app文件。一路 点击“OK”或者“YES” 就可以了。

打开界面后，双击左侧的 viewDidLoad：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602201058242-1824337591.png)

可以看出：代码中的  "CeShiLabel007" 字符串 完全可以反编译出来。所以尽量不要在代码里放一些 关键 的数据。可以通过接口来获取。或者 把 数据进行加密。

从上面的界面中 ，按下键盘的F5，可以 把汇编转成C语言代码。可读性很高。。

你如果试了 就发现你的F5不管用啊，那是因为 F5是一个插件Hex-Rays.Decompiler 的快捷键，这个插件是收费的、收费的。

 

##**六、反编译工具：Hopper Disassembler**

下载 Hopper Disassembler软件。打开ipa解压的.app。 或者直接 把.app拖进去。

双击“viewDidLoad”： 可以看到 汇编代码， "CeShiLabel007" 字符串、setText方法 等。

 

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602204404555-1739403755.png)

点击右上角的 if(a) f(x)图标：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602204528664-464088547.png)

会弹出 类似源代码的 伪编码：

 

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160602204639992-255404740.png)

 

代码中可以清楚的看到 处理的逻辑。简单易懂，和看源代码没太大区别。。

下面 是我从越狱手机的PP助手上下载的 微信 的ipa  进行反编译,看下里面的 QQContactInfoViewController 页面 的 viewDidLoad方法里的代码 怎么写的，

截图：

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160603132557742-293853908.png)

 

如果 你把从 AppStore下载的 ipa包直接拖到 IDA或Hopper里，看到的就是乱码，刚才已经说过了。AppStore的ipa是加过壳的 。如图：

 ![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160603131037805-1919649565.png)

 

##**七、抓包-https**

Charles 老版本和 新版本 抓取https 的配置 不一样。。

先看HTTP抓包：

1. 打开Charles程序
2. 查看Mac电脑的IP地址，如192.168.1.7
3. 打开iOS设置，进入当前wifi连接，设置HTTP代理Group，将服务器填为上一步中获得的IP，即192.168.1.7，端口填8888
4. iOS设备打开你要抓包的app进行网络操作
5. Charles弹出确认框，点击Allow按钮即可

HTTPS 老版本抓包：

1. 下载Charles证书http://www.charlesproxy.com/ssl.zip，解压后导入到iOS设备中（将crt文件作为邮件附件发给自己，再在iOS设备中点击附件即可安装；也可上传至dropbox之类的网盘，通过safari下载安装）
2. 在Charles的工具栏上点击设置按钮，选择Proxy Settings…
3. 切换到SSL选项卡，选中Enable SSL Proxying
4. 这一步跟Fiddler不同，Fiddler安装证书后就可以抓HTTPS网址的包了，Charles 还 需要在上一步的SSL选项卡的Locations表单填写要抓包的域名和端口，点击Add按钮，在弹出的表单中Host填写域名，比如填api.instagram.com，Port填443

HTTPS 新版本抓包：

　　Charles新版本 的 Proxy Settings 选项里是没有 SSL选项卡的。在左侧的域名上点右键：Enable SSL Proxying，就可以用了。

![img](http://images2015.cnblogs.com/blog/859442/201606/859442-20160603130151602-184733403.png)

 

然后 点击APP，会看到HTTPS解密的json数据。如果接口返回的数据 本身进行了加密，那你看到的还是乱码。

 

##**七、https - iOS 的代码如何写**

2015年4月末，网爆流行IOS网络通信库`AFNetworking SSL`漏洞，影响银联、中国银行、交通银行在内的2.5万个IOS应用，我来看下 各种网络写法对应的问题。

1、信任任何证书。在 AFNetworking 中 定义 allowInvalidCertificates 为true，表示 忽略所有证书。

```
AFHTTPRequestOperationManager * manager = [AFHTTPRequestOperationManager manager];

manager.securityPolicy.allowInvalidCertificates = YES;
```

 

这种情况下 用我们上面讲的方法，用Charles很容易 破解HTTPS加密的数据。

这种情况，一般是 因为 测试环境 用的不是 CA发的证书，需要忽略掉证书，所以把 allowInvalidCertificates 设为了 true。这个可以用 #ifdef DEBUG 来进行设置。

```
    #ifdef DEBUG
    manager.securityPolicy.allowInvalidCertificates = YES;
    #endif
```

 

2、信任证书管理机构（CA）颁发的证书。

CA颁发的证书，据说这类的证书只需50美元就能买到。此类问题出在`AFNetworking 2.5.2`及之前的版本，是AF的漏洞（[详见新闻](http://www.freebuf.com/news/65744.html)）。如果某IOS APP使用了此版本的开源通信库，在不安全Wifi网络中的，黑客 只要使用CA颁发的证书就可以对该APP的HTTPS加密数据进行监听或者篡改。

这个需要升级到 AFNetworking 最新版本，正好最新版本也声明兼容IPv6。

 

3、信任合法的证书、服务器和客户端双向认证。

这两种也都有办法破解，详见：[Bypassing OpenSSL Certificate Pinning in iOS Apps](https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/2015/january/bypassing-openssl-certificate-pinning-in-ios-apps/)、[http://drops.wooyun.org/tips/7838](http://drops.wooyun.org/tips/7838)

 

要正确的使用HTTPS才不会出现上面的问题。接口也一定要用自己的方式进行加密 才真正的放心，把小命完全放到对方(HTTPS)手里，命运就只能靠别人来摆布。。

