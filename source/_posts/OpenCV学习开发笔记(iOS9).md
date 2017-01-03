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

## 模块

下面是在官方文档中列出的最重要的模块。

core：简洁的核心模块，定义了基本的数据结构，包括稠密多维数组 Mat 和其他模块需要的基本函数。
imgproc：图像处理模块，包括线性和非线性图像滤波、几何图像转换 (缩放、仿射与透视变换、一般性基于表的重映射)、颜色空间转换、直方图等等。
video：视频分析模块，包括运动估计、背景消除、物体跟踪算法。
calib3d：包括基本的多视角几何算法、单体和立体相机的标定、对象姿态估计、双目立体匹配算法和元素的三维重建。
features2d：包含了显著特征检测算法、描述算子和算子匹配算法。
objdetect：物体检测和一些预定义的物体的检测 (如人脸、眼睛、杯子、人、汽车等)。
ml：多种机器学习算法，如 K 均值、支持向量机和神经网络。
highgui：一个简单易用的接口，提供视频捕捉、图像和视频编码等功能，还有简单的 UI 接口 (iOS 上可用的仅是其一个子集)。
gpu：OpenCV 中不同模块的 GPU 加速算法 (iOS 上不可用)。
ocl：使用 OpenCL 实现的通用算法 (iOS 上不可用)。
一些其它辅助模块，如 Python 绑定和用户贡献的算法。

## 基础类和操作

OpenCV 包含几百个类。为简便起见，我们只看几个基础的类和操作，进一步阅读请参考全部文档。过一遍这几个核心类应该足以对这个库的机理产生一些感觉认识。

cv::Mat

cv::Mat 是 OpenCV 的核心数据结构，用来表示任意 N 维矩阵。因为图像只是 2 维矩阵的一个特殊场景，所以也是使用 cv::Mat来表示的。也就是说，cv::Mat 将是你在 OpenCV 中用到最多的类。

一个 cv::Mat 实例的作用就像是图像数据的头，其中包含着描述图像格式的信息。图像数据只是被引用，并能为多个 cv::Mat 实例共享。OpenCV 使用类似于 ARC 的引用计数方法，以保证当最后一个来自 cv::Mat 的引用也消失的时候，图像数据会被释放。图像数据本身是图像连续的行的数组 (对 N 维矩阵来说，这个数据是由连续的 N-1 维数据组成的数组)。使用 step[] 数组中包含的值，图像的任一像素地址都可通过下面的指针运算得到：

```
uchar *pixelPtr = cvMat.data + rowIndex * cvMat.step[0] + colIndex * cvMat.step[1]
```
每个像素的数据格式可以通过 type() 方法获得。除了常用的每通道 8 位无符号整数的灰度图 (1 通道，CV_8UC1) 和彩色图 (3 通道，CV_8UC3)，OpenCV 还支持很多不常用的格式，例如 CV_16SC3 (每像素 3 通道，每通道使用 16 位有符号整数)，甚至CV_64FC4 (每像素 4 通道，每通道使用 64 位浮点数)。

cv::Algorithm

Algorithm 是 OpenCV 中实现的很多算法的抽象基类，包括将在我们的 demo 工程中用到的 FaceRecognizer。它提供的 API 与苹果的 Core Image 框架中的 CIFilter 有些相似之处。创建一个 Algorithm 的时候使用算法的名字来调用 Algorithm::create()，并且可以通过 get() 和 set()方法来获取和设置各个参数，这有点像是键值编码。另外，Algorithm 从底层就支持从/向 XML 或 YAML 文件加载/保存参数的功能。

# 实战部分

## 导入工程

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

把使用到 OpenCV 中 C++方法的实现文件后缀名改成.mm，就可以开始使用 OpenCV 的方法了。

### Objective-C++

如前面所说，OpenCV 是一个 C++ 的 API，因此不能直接在 Swift 和 Objective-C 代码中使用，但能在 Objective-C++ 文件中使用。

Objective-C++ 是 Objective-C 和 C++ 的混合物，让你可以在 Objective-C 类中使用 C++ 对象。clang 编译器会把所有后缀名为.mm 的文件都当做是 Objective-C++。一般来说，它会如你所期望的那样运行，但还是有一些使用 Objective-C++ 的注意事项。内存管理是你最应该格外注意的点，因为 ARC 只对 Objective-C 对象有效。当你使用一个 C++ 对象作为类属性的时候，其唯一有效的属性就是 assign。因此，你的 dealloc 函数应确保 C++ 对象被正确地释放了。

第二重要的点就是，如果你在 Objective-C++ 头文件中引入了 C++ 头文件，当你在工程中使用该 Objective-C++ 文件的时候就泄露了 C++ 的依赖。任何引入你的 Objective-C++ 类的 Objective-C 类也会引入该 C++ 类，因此该 Objective-C 文件也要被声明为 Objective-C++ 的文件。这会像森林大火一样在工程中迅速蔓延。所以，应该把你引入 C++ 文件的地方都用 #ifdef __cplusplus包起来，并且只要可能，就尽量只在 .mm 实现文件中引入 C++ 头文件。



看起来很简单，然而实际操作中还是有不少的问题。



<!------MORE------->


## 小热身

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

## Demo:人脸识别

Demo：人脸检测与识别

现在，我们对 OpenCV 及如何把它集成到我们的应用中有了大概认识，那让我们来做一个小 demo 应用：从 iPhone 的摄像头获取视频流，对它持续进行人脸检测，并在屏幕上标出来。当用户点击一个脸孔时，应用会尝试识别这个人。如果识别结果正确，用户必须点击 “Correct”。如果识别错误，用户必须选择正确的人名来纠正错误。我们的人脸识别器就会从错误中学习，变得越来越好。

### 视频拍摄


OpenCV 的 highgui 模块中有个类，CvVideoCamera，它把 iPhone 的摄像机抽象出来，让我们的 app 通过一个代理函数 - (void)processImage:(cv::Mat&)image 来获得视频流。CvVideoCamera 实例可像下面这样进行设置：

```objc
CvVideoCamera *videoCamera = [[CvVideoCamera alloc] initWithParentView:view];  
videoCamera.defaultAVCaptureDevicePosition = AVCaptureDevicePositionFront;  
videoCamera.defaultAVCaptureSessionPreset = AVCaptureSessionPreset640x480;  
videoCamera.defaultAVCaptureVideoOrientation = AVCaptureVideoOrientationPortrait;  
videoCamera.defaultFPS = 30;  
videoCamera.grayscaleMode = NO;  
videoCamera.delegate = self;
```

摄像头的帧率被设置为 30 帧每秒， 我们实现的 processImage 函数将每秒被调用 30 次。因为我们的 app 要持续不断地检测人脸，所以我们应该在这个函数里实现人脸的检测。要注意的是，如果对某一帧进行人脸检测的时间超过 1/30 秒，就会产生掉帧现象。

### 人脸检测

其实你并不需要使用 OpenCV 来做人脸检测，因为 Core Image 已经提供了 CIDetector 类。用它来做人脸检测已经相当好了，并且它已经被优化过，使用起来也很容易：

```objc
CIDetector *faceDetector = [CIDetector detectorOfType:CIDetectorTypeFace context:context options:@{CIDetectorAccuracy: CIDetectorAccuracyHigh}];

NSArray *faces = [faceDetector featuresInImage:image];
```

从该图片中检测到的每一张面孔都在数组 faces 中保存着一个 CIFaceFeature 实例。这个实例中保存着这张面孔的所处的位置和宽高，除此之外，眼睛和嘴的位置也是可选的。

另一方面，OpenCV 也提供了一套物体检测功能，经过训练后能够检测出任何你需要的物体。该库为多个场景自带了可以直接拿来用的检测参数，如人脸、眼睛、嘴、身体、上半身、下半身和笑脸。检测引擎由一些非常简单的检测器的级联组成。这些检测器被称为 Haar 特征检测器，它们各自具有不同的尺度和权重。在训练阶段，决策树会通过已知的正确和错误的图片进行优化。关于训练与检测过程的详情可参考此原始论文。当正确的特征级联及其尺度与权重通过训练确立以后，这些参数就可被加载并初始化级联分类器了：

```objc
/ 正面人脸检测器训练参数的文件路径
NSString *faceCascadePath = [[NSBundle mainBundle] pathForResource:@&quot;haarcascade_frontalface_alt2&quot;  
                                                   ofType:@&quot;xml&quot;];
 
const CFIndex CASCADE_NAME_LEN = 2048;  
char *CASCADE_NAME = (char *) malloc(CASCADE_NAME_LEN);  
CFStringGetFileSystemRepresentation( (CFStringRef)faceCascadePath, CASCADE_NAME, CASCADE_NAME_LEN);
 
CascadeClassifier faceDetector;  
faceDetector.load(CASCADE_NAME);
```

这些参数文件可在 OpenCV 发行包里的 data/haarcascades 文件夹中找到。

在使用所需要的参数对人脸检测器进行初始化后，就可以用它进行人脸检测了：

```objc
cv::Mat img;  
vector&lt;cv::Rect&gt; faceRects;  
double scalingFactor = 1.1;  
int minNeighbors = 2;  
int flags = 0;  
cv::Size minimumSize(30,30);  
faceDetector.detectMultiScale(img, faceRects,  
                              scalingFactor, minNeighbors, flags
                              cv::Size(30, 30) );
```

检测过程中，已训练好的分类器会用不同的尺度遍历输入图像的每一个像素，以检测不同大小的人脸。参数 scalingFactor 决定每次遍历分类器后尺度会变大多少倍。参数 minNeighbors 指定一个符合条件的人脸区域应该有多少个符合条件的邻居像素才被认为是一个可能的人脸区域；如果一个符合条件的人脸区域只移动了一个像素就不再触发分类器，那么这个区域非常可能并不是我们想要的结果。拥有少于 minNeighbors 个符合条件的邻居像素的人脸区域会被拒绝掉。如果 minNeighbors 被设置为 0，所有可能的人脸区域都会被返回回来。参数 flags 是 OpenCV 1.x 版本 API 的遗留物，应该始终把它设置为 0。最后，参数 minimumSize 指定我们所寻找的人脸区域大小的最小值。faceRects 向量中将会包含对 img 进行人脸识别获得的所有人脸区域。识别的人脸图像可以通过cv::Mat 的 () 运算符提取出来，调用方式很简单：cv::Mat faceImg = img(aFaceRect)。

不管是使用 CIDetector 还是 OpenCV 的 CascadeClassifier，只要我们获得了至少一个人脸区域，我们就可以对图像中的人进行识别了。


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
 



