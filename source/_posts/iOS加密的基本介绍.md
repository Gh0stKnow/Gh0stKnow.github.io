---
title: iOS加密的基本介绍
permalink: iosjia-mi-de-ji-ben-jie-shao
id: 32
updated: '2016-08-14 17:59:10'
date: 2016-08-08 13:52:13
tags:
---



安全隐患：在iOS开发过程中，尽管在发送数据的过程中，密码进行了“二进制”的转换，但实际上密码还是明文，可以通过一些拦截软件被拦截（青花瓷等等），不能保证我们数据的安全性。

加密选择：一般公司都会有一套自己的加密方案，按照公司接口文档的规定去加密

平常用到的解决办法：

# 1.Base64加密

> Base64加密因其算法和充当密钥的索引表都是公开的，所以不属于加密算法，它的本质是将“二进制”数据转换成字符串，方便使用HTTP协议、用于公开的代码加密、URL加密，防止数据明文传输。

例如：有的网络请求上，会希望只传递字符串

1.URL中的参数，直接带上图片的传输

2.银联的网络接口，把整个消费凭据生成一个数据格式，然后进行Base64的编码，编码完成后传给服务器

特点：编码完成后的结果，只有64个字符。

转换方法：

> 1.将每三个字节分成一组，一共24个二进制位：3*8=24
>
> 2.将这24个二进制位分成4组，每组有6个二进制位：24/4=6
>
> 3.在每组前加两00，扩展成32个二进制位，即4个字节：4*(6+2)=32
>
> 4.根据Base64索引表就可得到对应的符号值。

Base64索引表

下面来看看它的具体转换过程：

加密的具体过程图

苹果在iOS7之后自带Base64加密，具体方法如下：


<!------MORE------->

# 2.MD5加密

> MD5即Message-Digest Algorithm 5（信息-摘要算法5），用于确保信息传输完整一致。是计算机广泛使用的杂凑算法之一（又译摘要算法、哈希算法），主流编程语言普遍已有MD5实现。作用是让大容量信息在用数字签名软件签署私人密钥前被"压缩"成一种保密的格式（就是把一个任意长度的字节串变换成一定长的十六进制数字串）。
>
> MD5加密对输入信息生成唯一的128位散列值（32个字符），声称是不可逆的，但是目前可以通过以下网站解密[MD5解密网站](http://www.cmd5.com) ，主要运用在数字签名、文件完整性验证以及口令加密等方面。

先来看看基本的加密算法实现过程：

![img](http://upload-images.jianshu.io/upload_images/974583-36ecd58f885b2b01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不过在平常的开发中我们一般都会用到第三方：例如 ** Security 的NSString+Hash.h** 等等，主要原因稍后会介绍。

基本使用方法：

![img](http://upload-images.jianshu.io/upload_images/974583-2d1695260849a0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为MD5生成的简单字符串会到目前会被轻易的破解，所以我们需要对其进行改进。

1.**\*加盐（Salt）***：在明文的固定位置插入随机串，然后再进行MD5

![img](http://upload-images.jianshu.io/upload_images/974583-54897bb397997310.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注：加的盐必须足够长，足够保密或者可以计入时间戳

如果还不行的话可以使用辅助手段 IP记录   手机短信记录   操作异常或者比较敏感的操作等手法。

2.**\*HMAC+MD5***：HMAC本来就是一种加密算法，再加上MD5，算得上双重保障

Security第三方有此方法

3.**先加密，后乱序**：先对明文进行MD5，然后对加密得到的MD5串的字符进行乱序，总之宗旨就是：黑客就算攻破了数据库，也无法解密出正确的明文

> 使用MD5的好处：快速校验

# 3. 对于重要数据，使用RSA进行数字签名，起到防篡改作用

# 4 .对于比较敏感的数据，如用户信息（登陆、注册等），客户端发送使用RSA加密，服务器返回使用DES(AES)加密

原因：客户端发送之所以使用RSA加密，是因为RSA解密需要知道服务器私钥，而服务器私钥一般盗取难度较大；

# 5.钥匙串（keychain）

iOS 7.0.3版本后加入，使用256位AES加密

> keychain Services：（iOS密钥链服务）提供了针对用户设备上的密码、密钥、证书、笔记和自定义数据的安全存储解决方案。保存的信息不会因App被删除而丢失，在用户重新安装App后依然有效，数据还在，是目前在设备中保存关键数据的唯一安全的地方。

那么，如何在应用里使用keychain呢？

我们需要导入Security.framework ，keychain的操作接口声明在头文件SecItem.h里，直接使用SecItem.h里方法操作keychain，需要写的代码较为复杂，我们可以使用已经封装好了的工具类KeychainItemWrapper来对keychain进行操作。KeychainItemWrapper是apple官方例子“GenericKeychain”里一个访问keychain常用操作的封装类，在官网上下载了GenericKeychain项目后，**只需要把“KeychainItemWrapper.h”和“KeychainItemWrapper.m”拷贝到我们项目，并导入Security.framework 。同时需要设置ARC（-fno-objc-arc）。**

我们来看看使用方法：

保存数据

取出数据

其他具体的使用方法，自己看API。

第三方封装的有**SSKeychain**，使用方法更为简便。

![img](http://upload-images.jianshu.io/upload_images/974583-6949590f265968a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![img](http://upload-images.jianshu.io/upload_images/974583-a16cfe95dc2702ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 6.Cookie（小甜饼）

由服务器生成，发送给客户端，将cookie的key/value保存到某个目录的文本文件内。

最典型的应用是判定注册用户是否已经登录，另一个重要的应用场合是“购物车”。

简单使用方法：

![img](http://upload-images.jianshu.io/upload_images/974583-ff4a1b563a0f2ddb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


原文链接：http://www.jianshu.com/p/2beffa24e889

