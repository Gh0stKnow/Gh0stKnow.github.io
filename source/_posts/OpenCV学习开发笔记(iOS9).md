---
title: OpenCV学习开发笔记(iOS9)
permalink: OpenCVLearning
date: 2016-12-29 09:52:38
tags: [学习笔记, 人脸识别]
---

本文章采用的的开发环境为：
1）Xcode 8.2
2）OpenCV for iOS 3.2 

# 前言
最近公司项目进入了较为稳定的维护周期，考虑到后面很可能会进行需要生物特征识别的项目，提前学习下OpenCV，也在此和大家分享一下。


# OpenCV介绍
OpenCV ，是一个开源的跨平台计算机视觉和机器学习库，通俗点的说，就是他给计算机提供了一双眼睛，一双可以从图片中获取信息的眼镜，从而完成人脸识别、去红眼、追踪移动物体等等的图像相关的功能。更多具体的说明可参见 OpenCV 官网。

# 导入工程

首先下载从官网[OpenCV官网](http://opencv.org/)下载的iOS支持库，我们新建一个工程。

![搭建环境](http://oiu3ghos7.bkt.clouddn.com/14829768918916.jpg)

导入 OpenCV 到 Xcode 的工程中还是比较简单的，从官网下载对应的 framework，直接丢到 Xcode 的工程中，从xcode7以后拖入的工程会自动添加到Building phase里面，检查一下。
![](http://oiu3ghos7.bkt.clouddn.com/14829783173349.jpg)

然后在你想用 OpenCV 的地方引入 OpenCV 的头文件：

```objc
#import <opencv2/opencv.hpp>

```    

或者直接在 PCH 文件中添加：  
  
```objc
#ifdef __cplusplus
#import <opencv2/opencv.hpp>
#endif

```    

<!----- MORE ----->

*关于这个宏下面会有解释说明*

然后把使用到 OpenCV 中 C++方法的实现文件后缀名改成.mm，就可以开始使用 OpenCV 的方法了。

看起来很简单，然而实际操作中还是有不少的问题。

# 实际使用

由于OpenCV代码是基于C++编写的，因此，要在项目中运行c++代码，需要将实现文件名后缀由`.m`改成`.mm`，如下图所示。
![](http://oiu3ghos7.bkt.clouddn.com/14829787278230.jpg)

再次强调一次使用opencv的类名一定要改成.mm!


说了那么多先测试一下吧!


```objc
#import <opencv2/opencv.hpp>
#import <opencv2/imgproc/types_c.h>
#import <opencv2/imgcodecs/ios.h>

#import "ViewController.h"


@interface ViewController ()
{
     cv::Mat cvImage;
}

@property (weak, nonatomic) IBOutlet UIImageView *imgView;


@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    UIImage *image = [UIImage imageNamed:@"learn.jpg"];
    
    UIImageToMat(image, cvImage);
    
    if(!cvImage.empty()){
        cv::Mat gray;
        // 将图像转换为灰度显示
        cv::cvtColor(cvImage,gray,CV_RGB2GRAY);
        // 应用高斯滤波器去除小的边缘
        cv::GaussianBlur(gray, gray, cv::Size(5,5), 1.2,1.2);
        // 计算与画布边缘
        cv::Mat edges;
        cv::Canny(gray, edges, 0, 50);
        // 使用白色填充
        cvImage.setTo(cv::Scalar::all(225));
        // 修改边缘颜色
        cvImage.setTo(cv::Scalar(0,128,255,255),edges);
        // 将Mat转换为Xcode的UIImageView显示
        self.imgView.image = MatToUIImage(cvImage);
    }
}

@end

```


原图
![finn-w290](http://oiu3ghos7.bkt.clouddn.com/finn.jpg)

运行在模拟器上的效果
![运行效果](http://oiu3ghos7.bkt.clouddn.com/14829830035512.jpg)




# 遇到问题
这是第一次导入C++的库进工程中，所以还是摸索了一段时间。

1. 第一次编译的时候遇到一个问题，编译器报了个警告是
```
#if defined(NO)
#  warning Detected Apple 'NO' macro definition, it can cause build conflicts. Please, include this header before any Apple headers.
#endif
```

按照提示说明，OpenCV的头文件应该在所有APPLE的头文件之前导入，不然会抛出异常，把import调到最前面即可。


2. 为何在 PCH 文件中引入 OpenCV 的头文件我们需要多加`#ifdef __cpluseplus`这一部分呢？这是因为 PCH 文件是一个会被**所有的文件**引入的头文件，而我们又希望 `#import <opencv2/opencv.hpp>`这部分只会被一些 C++实现文件编译，所以我们加上`#ifdef __cpluseplus`来表示这是 `C++` 文件才会编译的，除了`#ifdef __cpluseplus`，还有`#ifdef __OBJC__`这样的宏来说明编译规则（按照 OC 文件编译），这样的宏多出现于一些会被多种类型的实现文件引用的头文件中。

3. 另外注意另一个问题：如果一个头文件是C++类型的头文件，那么一定要保证所有`直接`或者`间接`引用这个头文件的实现文件都要为.mm或者.cpp，否则 Xcode 就不会把这个头文件当做 C++头文件来编译，就会出现最基本的`#include <iostream>`这种引用都会报出`file not found `这样的编译错误的问题。我在编译的过程中，某个C++头文件 A.h 被 B.h 引用，然后 B.h 又被 C.m 引用，虽然 B 的实现文件是 B.mm ，但是仍然报出了之前说的那个错误
感谢 StackOberflow 让我找到了问题发生的原因。所以对于 C++ 头文件的引用一定要注意，但凡是引用了 A.h 的实现部分，都必须是.mm或者.cpp后缀名。（同时我们也可以知道，Xcode 是根据头文件被引用的情况来判定头文件的编译 类型的）。






# 转换UIImage和cv::Mat

在 OpenCV 中同常用 cv::Mat 表示图片，而 iOS 中则是 UIImage 来表示图片，但openCV的[官方教程](http://docs.opencv.org/2.4/doc/tutorials/ios/image_manipulation/image_manipulation.html#opencviosimagemanipulation)已经给出了方法。

## UIImage to cv::Mat

```objc
    - (cv::Mat)cvMatFromUIImage:(UIImage *)image
{
  CGColorSpaceRef colorSpace = CGImageGetColorSpace(image.CGImage);
  CGFloat cols = image.size.width;
  CGFloat rows = image.size.height;

  cv::Mat cvMat(rows, cols, CV_8UC4); // 8 bits per component, 4 channels (color channels + alpha)

  CGContextRef contextRef = CGBitmapContextCreate(cvMat.data,                 // Pointer to  data
                                                 cols,                       // Width of bitmap
                                                 rows,                       // Height of bitmap
                                                 8,                          // Bits per component
                                                 cvMat.step[0],              // Bytes per row
                                                 colorSpace,                 // Colorspace
                                                 kCGImageAlphaNoneSkipLast |
                                                 kCGBitmapByteOrderDefault); // Bitmap info flags

  CGContextDrawImage(contextRef, CGRectMake(0, 0, cols, rows), image.CGImage);
  CGContextRelease(contextRef);

  return cvMat;
}

```


## cv::Mat to UIImage 

```objc
-(UIImage *)UIImageFromCVMat:(cv::Mat)cvMat
{
  NSData *data = [NSData dataWithBytes:cvMat.data length:cvMat.elemSize()*cvMat.total()];
  CGColorSpaceRef colorSpace;

  if (cvMat.elemSize() == 1) {//可以根据这个决定使用哪种
      colorSpace = CGColorSpaceCreateDeviceGray();
  } else {
      colorSpace = CGColorSpaceCreateDeviceRGB();
  }

  CGDataProviderRef provider = CGDataProviderCreateWithCFData((__bridge CFDataRef)data);

  // Creating CGImage from cv::Mat
  CGImageRef imageRef = CGImageCreate(cvMat.cols,                                 //width
                                     cvMat.rows,                                 //height
                                     8,                                          //bits per component
                                     8 * cvMat.elemSize(),                       //bits per pixel
                                     cvMat.step[0],                            //bytesPerRow
                                     colorSpace,                                 //colorspace
                                     kCGImageAlphaNone|kCGBitmapByteOrderDefault,// bitmap info
                                     provider,                                   //CGDataProviderRef
                                     NULL,                                       //decode
                                     false,                                      //should interpolate
                                     kCGRenderingIntentDefault                   //intent
                                     );


  // Getting UIImage from CGImage
  UIImage *finalImage = [UIImage imageWithCGImage:imageRef];
  CGImageRelease(imageRef);
  CGDataProviderRelease(provider);
  CGColorSpaceRelease(colorSpace);

  return finalImage;
 }
 
 ```
 



