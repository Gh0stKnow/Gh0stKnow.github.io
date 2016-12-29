---
title: Swift 中的异常抛出和错误处理
tags: swift
permalink: swift-zhong-de-yi-chang-pao-chu-he-cuo-wu-chu-li
id: 23
updated: '2016-06-28 12:20:14'
date: 2016-06-28 12:19:24
---


# 前言
Swift语言对其错误处理进行了新的设计，当然了，重新设计后的结果使得该错误处理系统用起来更爽。今天的主题就是系统的搞一下Swift中的错误处理，以及看一下Swift中是如何抛出异常的。在编译型语言中，错误一般分为编译错误和运行时错误。我们平时在代码中处理的错误为运行时错误，我们对异常进行处理的操作的目的是为了防止程序出现错误而导致其他的副作用，比如用户数据未保存等等。

在今天的文章中，先给出主动产生异常的几种情况，然后再给出如何处理被动异常。

# 问题总结

**一、主动退出程序的几种情况**

在Objective-C中，在单元测试时我们会使用断言，断言中条件满足时会产生异常，并打印出相应的断言错误，在Swift中也有几种产生异常的语法。在本篇博客的第一部分就给出这几种方法。

**1.Fatal Errors（致命的错误）**

使用fatalError()函数可以立即终止你的应用程序，在fatalError()中可以给出终止信息。使用fatalError()函数，会毫无条件的终止你的应用程序，用起来也是比较简单的，就是一个函数的调用。下方这个Demo一目了然呢，在此就不做过多赘述了。

![img](http://files.jb51.net/file_images/article/201602/201602261147512.png)

**2. Assertions（断言）**

在单元测试中是少不了断言的，Swift中的断言和Objective-C的区别不是太大，使用方法也是大同小异。下方就是断言的两种方法，由代码提示可知，在断言中的提示条件是可选的。断言会在Debug模式下起作用，但是在Release版本中就会被忽略。

![img](http://files.jb51.net/file_images/article/201602/201602261147513.png)

在assert()函数中, 第一个参数是Bool类型，第二个参数是输出的信息。当条件为true时，断言不执行，相应的断言信息不打印。当条件为false时，断言执行，并且打印相应的断言信息。

![img](http://files.jb51.net/file_images/article/201602/201602261147514.png)

assertionFailure()函数只有一个Message参数，并且该参数也是可以省略的，assertionFailure()没有条件。如下所示：

![img](http://files.jb51.net/file_images/article/201602/201602261147515.png)

**3. 先决条件（Preconditions）**

Preconditions的用法和断言一样，不过有一点需要主要，Preconditions在debug和release模式下都会被执行，除非使用–Ounchecked进行编译。下方截图是代码提示给出的Preconditions函数的提示，如下所示：

![img](http://files.jb51.net/file_images/article/201602/201602261147516.png)

关于Preconditions的具体用法请参照断言，和断言用法一样，在此就不做过多的赘述了。

**二.Swift中的错误处理**

在Objective-C中，如果你处理过错误的话，那么你将会对NSError很熟悉。在Swift中，如果你要定义你自己的错误类型，你只需要实现ErrorType协议即可。声明完错误类型后，就可以在处理错误抛出异常时使用自定义的错误类型了。下方将会一步步带你走完Swift中的错误处理的路程。

**1.使用枚举创建错误类型**

（1）.遵循ErrorType协议，自定义错误类型。下方定义了一个错误类型枚举，该枚举遵循了ErrorType协议，在接下来的代码中我们将会使用这个MyCustomErrorType枚举，错误枚举的实现如下所示：

```
 //定义错误类型
 enum MyCustomErrorType: ErrorType {
 case ErrorReason
 case ErrorReason
 case ErrorReason
 } 
```

（2）.在我们的函数定义时可以使用throws关键字，以及在函数中使用throw关键字对错误进行抛出，抛出的错误类型就可以使用上面我们自己定义的错误类型。下方函数就是一个可以抛出错误的函数，抛出的错误就是我们在上面枚举中所定义的类型。具体代码如下所示：

```
 func myThrowFunc() throws {
 let test:Int? = nil
 guard test != nil else {
 throw MyCustomErrorType.ErrorReason
 }
 }
```

（3）.上面函数的功能是对错误进行抛出，接下来就该使用do-catch来处理抛出的错误。使用try对错误进行捕捉，使用do-catch对错误进行处理。具体处理方式如下所示。在下方错误处理中类似于switch-case语句，catch后边可以枚举匹配错误类型，具体如下所示：　　　　

![img](http://files.jb51.net/file_images/article/201602/201602261147517.png)

（4）在枚举实现错误类型中我们可以通过值绑定的形式为错误添加错误代码和错误原因。在声明枚举时，我们使用了枚举元素值绑定的特性（关于枚举使用的更多细节请参考之前的博客《窥探Swift之别样的枚举类型》）。在声明枚举成员ErrorState时，我们为其绑定了两个变量，一个是错误代码errorCode, 另一个是错误原因errorReason。这两者可以在抛出错误时为其传入相应的值，如下方代码片段中的throwError函数所示，在抛出错误是为errorCode指定的错误代码为404，为errorReason指定的错误原因是“not found”。

　　最后就是使用do-catch处理异常了，在catch中对绑定的错误代码和错误原因进行了获取，并且通过where子句进行了错误代码的筛选。此处catch的用法与switch-case中获取枚举绑定值的用法是一样的，所以在此就不做过多的赘述。具体实现方式如下代码所示：

![img](http://files.jb51.net/file_images/article/201602/201602261147518.png)

**2.使用结构体为错误处理添加Reason**

在上面的内容中，使用枚举遵循ErrorType协议的方式定义了特定的错误类型。接下来我们将使用结构体来遵循ErrorType协议，为错误类型添加错误原因。也就是说，我们可以在抛出错误时，给自定义错误类型提供错误原因。该功能在开发中是非常常用的，而且用起来也是非常爽的。接下来就看一下如何为我们的错误类型添加错误原因。

（1）使用结构体创建错误类型，下方名为MyErrorType的结构体遵循了ErrorType协议，并且在MyErrorType结构体中，声明了一个reason常量，该reason常量中存储的就是错误原因，具体实现方式如下：

```
 struct MyErrorType: ErrorType {
 let reason : String
 }
```

（2）上面定义完错误类型结构体后，在错误抛出中就可以使用了。在错误抛出时，可以传入一个错误原因，具体代码如下所示：

```
 func myThrowFunc() throws {
 let test:Int? = nil
 guard test != nil else {
 throw MyErrorType(reason: "我是详细的错误原因，存储在error中")
 }
 } 
```

（3）最后要对抛出的错误进行do-catch处理，在处理时，可以对错误原因进行打印，错误原因存储在error中，具体操作和打印结果如下所示：　　　　　　

![img](http://files.jb51.net/file_images/article/201602/201602261147519.png)

由上面的输出结果可知，error是我们自定义的MyErrorType类型，我们可以使用下面的代码来代替catch中的print语句，如下所示：　　　　

![img](http://files.jb51.net/file_images/article/201602/2016022611475110.png)

上面的做法似乎有些麻烦，还有一种简化输出的方法，就是在上述结构体中实现CustomDebugStringConvertible协议，对描述信息进行一个重写，就可以在打印error时，只打印错误信息，下方是重写后的结构体。　　　

```
 struct MyErrorType: ErrorType,CustomDebugStringConvertible {
 let reason : String
 var debugDescription: String {
 return "错误类型-----\(self.dynamicType): \(reason)"
 }
 } 
```

修改后，输出结果如下，直接打印error输出的就是错误信息，而不是MyErrorType类型。

![img](http://files.jb51.net/file_images/article/201602/2016022611475111.png)

**3.使String类型遵循ErrorType协议，直接使用String提供错误原因**

在“2”中，我们使用了结构体遵循ErrorType协议的形式，来为错误提供错误信息的。在接下来的部分，我们将通过更为简单的方式为抛出的错误提供错误信息。这种方式更为简单，也易于理解，具体方式如下方代码所示：　　　　

![img](http://files.jb51.net/file_images/article/201602/2016022611475112.png)

**三、在错误处理中使用内置关键字**

**1.初探这些内置关键字**

在Swift中提供了一些内置关键字（__FILE__, __FUNCTION__, __LINE__等）来获取上下文信息，在本篇博客的第三部分，将会给出如何在我们的错误处理中使用这些内置关键字。下方就是这些内置关键字的作用，如下所示：

![img](http://files.jb51.net/file_images/article/201602/2016022611475113.png)

上面说是内置关键字，其实就是存储代码上下文的宏定义，上方代码段简单的给出了这些内置关键字的作用与用法，在接下来将在ErrorType中使用这些内置关键字，让我们的错误信息更加丰富多彩。　

**2.在ErrorType中使用上述内置关键字**

如果想在ErrorType中使用这些上下文内置关键字，我们只需要对ErrorType进行扩展，使其在ErrorType提供错误信息时给出出错的上下文信息。当然，这实现起来比较简单，就是在ErrorType中添加了一个扩展方法contextString()。该方法的作用就是提供错误的上下文信息，也就是在出错的地方，调用contextString()方法生成上下文描述信息即可。对ErrorType协议的具体延展实现如下代码段所示.

在下方代码片段中，我们对ErrorType进行了扩展，为ErrorType添加了contextString的函数实现。contextString()函数有三个默认参数，分别是file--当前文件名，function--当前出错的函数名，line--当前抛出异常的行数。上述三个参数都有参数默认值，分别对应着__FILE__, __FUNCTION__, __LINE__。该扩展函数的返回值为这三个参数组成从字符串信息。具体实现如下所示：

![img](http://files.jb51.net/file_images/article/201602/2016022611475214.png)

**3.使用扩展的contextString方法**

上面我们使用结构体实现ErrorType协议的形式，为错误类型添加错误原因。接下来我们将在添加reason的同时，使用contextString()函数添加描述信息。下方CustomErrorType结构体遵循了ErrorType协议，其中添加了一个reason常量来存储错误原因，一个context常量来存储上下文信息，并且为该结构体添加了一个构造函数，在构造函数中初始化和reason常量。具体实现如下所示：　　

![img](http://files.jb51.net/file_images/article/201602/2016022611475215.png)

**4. 抛出并捕获异常**

在下方代码中函数throwError()抛出了异常，该抛出的错误类型是CustomErrorType。在创建CustomErrorType类型实例，也就是err变量时，我们指定了错误原因，也就是为reason赋了一个值。在创建完err实例后，我们又调用延展contextString()函数获取异常的上下文信息，并把返回的内容存储在err实例的context属性中。最后使用throw关键字抛出err实例，如下方第一部分代码所示。

在创建抛出异常的函数后，我们需要对抛出的异常进行捕获。也就是使用try对异常进行捕获，使用do-catch对异常进行处理，具体操作如下方第二段代码所示。

　　　![img](http://files.jb51.net/file_images/article/201602/2016022611475216.png)　

**5. 分析打印结果**

经过上述步骤如果你在Playground中进行试验的，那么在控制台上你将会看到如下信息。从打印出的信息我们可以看到，信息包括reason：错误原因，和context：异常上下文。在下方的输出结果中，文件名我们可以看到是<EXPR>这并不是确切的文件名，因为我们是在Playground中使用的，并且不是确切的Swift源文件，所以获取不到确切的文件名。

![img](http://files.jb51.net/file_images/article/201602/2016022611475217.png)

为了观察确切的文件名，我们需要在确切的Swift源文件中抛出上述异常。在特定Swift源文件中，我们会看到下方的输出结果。从下方的输出日志中，我们可以清楚的看到文件名是一个详细的文件路径。如下所示：　　　　

![img](http://files.jb51.net/file_images/article/201602/2016022611475218.png)

