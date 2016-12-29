---
title: 基于 ijkplayer 的iOS视频直播推流仿写
tags: ' - 转载'
permalink: osshi-pin-zhi-bo-fang
id: 27
updated: '2016-08-11 00:05:29'
date: 2016-07-07 11:39:36
---



视频直播，可以分为 采集，前处理，编码，传输, 服务器处理，解码，渲染

采集: iOS系统因为软硬件种类不多, 硬件适配性比较好, 所以比较简单. 而Android端市面上机型众多, 要做些机型的适配工作.PC端是最麻烦的, 各种奇葩摄像头驱动.所以现在很多的中小型直播平台, 都放弃了PC的直播, 更有一些直播平台只做iOS端的视频直播.

前处理: 美颜算法,视频的模糊效果, 水印等都是在这个环节做. 目前iOS端最著名开源框架的毫无疑问就是GPUImage.其中内置了125种渲染效果, 还支持各种脚本自定义. 我高仿的喵播的美颜效果也是基于GPUImage的.

编码: 重难点在于要在分辨率，帧率，码率，GOP等参数设计上找到最佳平衡点。iOS8之后, Apple开放了VideoToolbox.framework, 可以直接进行硬编解码, 这也是为什么现在大多数直播平台最低只支持到iOS8的原因之一. iOS端硬件兼容性比较好, 可以直接采取硬编码. 而Android得硬编码又是一大坑.

传输: 这块一般都是交给CDN服务商. CDN只提供带宽和服务器之间的传输, 发送端和接收端的网络连接抖动缓存还是要自己实现的.目前国内最大的CDN服务商应该是网宿.

服务器处理: 需要在服务器做一些流处理工作, 让推送上来的流适配各个平台各种不同的协议, 比如:RTMP,HLS,FLV...

解码和渲染: 也就即音视频的播放. 解码毫无疑问也必须要硬解码. iOS端兼容较好, Android依然大坑.这块的难点在于音画同步, 目前很多直播平台这块是硬伤.国内比较好的开源项目应该是B站开源的ijkplayer . 斗鱼就是基于ijkplayer 的, 本项目也是基于ijkplayer 的.

技术坑 : 降噪, 音频解码器, 蓝牙适配, 回声消除, 信令控制, 登录, 鉴权, 权限管理, 状态管理, 应用消息, 消息推送, 礼物系统, 即时聊天, 支付系统, 统计系统, 数据库, 缓存, 分布式文件存储, 消息队列, 运维系统等等大小不一的坑等你来填!!!


<!------MORE------->


资金坑 : 以带宽为例, 2万人同时在线, 手机码率在600KB, 每个月的带宽费用至少在30万左右. 根据欢聚时代(YY)15年四季度财务报, 他们的带宽成本为人民币1.611亿元, 折合每月5000万+. 人力成本+渠道支出和其他支出就不详谈了.

社会坑: 还得每时每刻与各种黑暗势力斗争, 包括色情, 广告, 刷小号, 刷充值, 告侵权, DDos...


前期准备
项目主要是基于ijkplayer 的. 最好是打包成framework. 原本我准备写一个打包教程, 不过后来在简书上发现了一篇特别详细的打包blog, 分享给大家: [http://www.jianshu.com/p/1f06b27b3ac0](http://)

Profile : 个人中心, 这里面只有一个ProfileController. 因为总写重复代码, 都写吐了, 这儿有兴趣的自己写一下吧, So easy...

Network : 关于网络连接的工具类. 关于网络的实时监控, 网络状态的切换, 网络请求的工具类都在这里面.

Other : 全局的常量. 当然你也可以在里面将文件结构更加细化.

Home : 包含最新主播, 最热直播, 关注的直播, 礼物排行榜等模块. 还有最重要的视频直播也在这里面了.

ShowTime :见名知意. 视频直播的前处理, 智能美颜和H264硬编码等都在这里面.

Main : UITabBarController和UINavigationController的配置

Toos : 这儿命名有点不规范, 这里面放置的都是项目用到的分类

Login : 登录模块

Resource : 项目用到的资源文件

项目详解
tip1: 判读网络类型.

在观看直播的时候, 我们通常都是用WiFi或者3/4G(土豪级别的), 一般用户在进行网络切换的时候, 我们都要给出友善的提示, 告诉TA: 您的网络状态切换到了XX状态. 假设用户从WiFi切换到4G, 你的应用也没个提醒, 导致TA的流量归零甚至欠了运营商一屁股的钱, 我想你的APP的用户体验也就归零或者为负了.
我们可以使用苹果的Reachability结合下面的代码实时监听网络状态的改变
```
typedef NS_ENUM(NSUInteger, NetworkStates) {
    NetworkStatesNone, // 没有网络
    NetworkStates2G, // 2G
    NetworkStates3G, // 3G
    NetworkStates4G, // 4G
    NetworkStatesWIFI // WIFI
};
// 判断网络类型
+ (NetworkStates)getNetworkStates
  {
    NSArray *subviews = [[[[UIApplication sharedApplication] valueForKeyPath:@"statusBar"] valueForKeyPath:@"foregroundView"] subviews];
    // 保存网络状态
    NetworkStates states = NetworkStatesNone;
    for (id child in subviews) {
        if ([child isKindOfClass:NSClassFromString(@"UIStatusBarDataNetworkItemView")]) {
            //获取到状态栏码
            int networkType = [[child valueForKeyPath:@"dataNetworkType"] intValue];
            switch (networkType) {
                case 0:
                   //无网模式
                    states = NetworkStatesNone;
                    break;
                case 1:
                    states = NetworkStates2G;
                    break;
                case 2:
                    states = NetworkStates3G;
                    break;
                case 3:
                    states = NetworkStates4G;
                    break;
                case 5:
                {
                    states = NetworkStatesWIFI;
                }
                    break;
                default:
                    break;
            }
        }
    }
    //根据状态选择
    return states;
  }
  tip2: 登录模块
```
如果你多运行几次就会发现, 登录模块背景中播放的视频是2个视频每次随机播放一个的.并且是无限重复的, 也就是说只要你一直呆着登录界面, 就会单视频循环播放当前的视频. 这儿的登录只是几个按钮, 没有具体的登录逻辑, 随便点哪一个按钮都可以进入首页.
我们需要监听视频, 是否播放完成.
```
// 监听视频是否播放完成
  [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(didFinish) name:IJKMPMoviePlayerPlaybackDidFinishNotification object:nil];
如果播放完成了, 让IJKFFMoviePlayerController再次play即可

- (void)didFinish
  {
    // 播放完之后, 继续重播
    [self.player play];
  }
  tip3: 首页
```
首页

这种效果相信很多人都看到过或者做过.我简单说一下我的做法(不一定是最佳的, 只是提供一个思路)

一个父控制器HomeViewController+三个子控制器(最热/最新/关注. 每个控制器各自管理自己的业务逻辑, 高内聚低耦合). 重写HomeViewController的loadView, 将self.view替换成UIScrollView. 将三个子控制器的view添加到UIScrollView上即可. 其他的效果实现, 请参照我的代码, 都有详细的中文注释.
tip4: 直播(面向观众端)
这个是整个项目的重点之一了.这种直播的布局, 应该是比较主流的了. 我下载的好多直播类APP都是这个项目布局, 包括YY也是这种界面布局.这个里面涉及的东西比较多了, 三言两语真说不清.

简单说一下已经实现的效果:
A: 主播的直播
B: 关联主播的视频直播, 默认是只有界面, 没有声音的. 点击该视图可以切换到此主播
C: 下拉切换另一个主播, 这个功能是很常见的. 做法是直播控制器是一个UICollectionViewController, 只有一个cell, 且cell.frame就是self.collectionViewb.bounds. 我们进入直播控制器的时候, 其实是传进去一个关联主播数组, 每次下拉的时候, 就加载数组里面的主播
D. 查看观众席的观众详情
E. 查看主播详情
F. 足迹: 粒子动画, 后面详解
G. 弹幕: 点击最下方的工具栏第一个按钮可以开启/关闭弹幕, 后面详解
...
tip5: 粒子动画实现游客足迹
粒子动画的layer是添加到播放器的view上面的. 下面代码有详细的注释
```
CAEmitterLayer *emitterLayer = [CAEmitterLayer layer];
// 发射器在xy平面的中心位置
emitterLayer.emitterPosition = CGPointMake(self.moviePlayer.view.frame.size.width-50,self.moviePlayer.view.frame.size.height-50);
// 发射器的尺寸大小
emitterLayer.emitterSize = CGSizeMake(20, 20);
// 渲染模式
emitterLayer.renderMode = kCAEmitterLayerUnordered;
// 开启三维效果
//    _emitterLayer.preservesDepth = YES;
NSMutableArray *array = [NSMutableArray array];
// 创建粒子
for (int i = 0; i<10; i++) {
    // 发射单元
    CAEmitterCell *stepCell = [CAEmitterCell emitterCell];
    // 粒子的创建速率，默认为1/s
    stepCell.birthRate = 1;
    // 粒子存活时间
    stepCell.lifetime = arc4random_uniform(4) + 1;
    // 粒子的生存时间容差
    stepCell.lifetimeRange = 1.5;
    // 颜色
    // fire.color=[[UIColor colorWithRed:0.8 green:0.4 blue:0.2 alpha:0.1]CGColor];
    UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"good%d_30x30", i]];
    // 粒子显示的内容
    stepCell.contents = (id)[image CGImage];
    // 粒子的名字
    //            [fire setName:@"step%d", i];
    // 粒子的运动速度
    stepCell.velocity = arc4random_uniform(100) + 100;
    // 粒子速度的容差
    stepCell.velocityRange = 80;
    // 粒子在xy平面的发射角度
    stepCell.emissionLongitude = M_PI+M_PI_2;;
    // 粒子发射角度的容差
    stepCell.emissionRange = M_PI_2/6;
    // 缩放比例
    stepCell.scale = 0.3;
    [array addObject:stepCell];
}

emitterLayer.emitterCells = array;
[self.moviePlayer.view.layer insertSublayer:emitterLayer below:self.catEarView.layer];
tip6: 弹幕
弹幕使用的也是一个第三方轮子BarrageRenderer . 这个开源项目的文档都是中文的, 用法也是很简单的.

基本配置

_renderer = [[BarrageRenderer alloc] init];
// 设置弹幕的显示区域. 基于父控件的.
_renderer.canvasMargin = UIEdgeInsetsMake(ALinScreenHeight * 0.3, 10, 10, 10);
[self.contentView addSubview:_renderer.view];
弹幕配置

#pragma mark - 弹幕描述符生产方法
/// 生成精灵描述 - 过场文字弹幕
- (BarrageDescriptor *)walkTextSpriteDescriptorWithDirection:(NSInteger)direction
  {
    BarrageDescriptor * descriptor = [[BarrageDescriptor alloc]init];
    descriptor.spriteName = NSStringFromClass([BarrageWalkTextSprite class]);
    descriptor.params[@"text"] = self.danMuText[arc4random_uniform((uint32_t)self.danMuText.count)];
    descriptor.params[@"textColor"] = Color(arc4random_uniform(256), arc4random_uniform(256), arc4random_uniform(256));
    descriptor.params[@"speed"] = @(100 * (double)random()/RAND_MAX+50);
    descriptor.params[@"direction"] = @(direction);
    descriptor.params[@"clickAction"] = ^{
        UIAlertView *alertView = [[UIAlertView alloc]initWithTitle:@"提示" message:@"弹幕被点击" delegate:nil cancelButtonTitle:@"取消" otherButtonTitles:nil];
        [alertView show];
    };
    return descriptor;
  }
  最后一步, 千万要记得start

[_renderer start];
```

tip7: 智能美颜效果
现在的直播平台, 美颜是标配. 不然绝大多数的主播都是没法看的.美颜算法需要用到GPU编程, 需要懂图像处理的人. 图像处理这一块我不是很熟悉, 相关的文献也是看得云里雾里的. 所以, 依然使用开源的轮子: GPUImage . 这个开源框架有近1.3W+star(7月5日数据), 真不是盖的, 内置125种滤镜效果, 没有你想不到, 只有你不会用. 我的项目中都有详细的用法, 还是很简单的. 在这里摘抄一份其.h文件的注释. 一方面方便大家修改我项目中的美颜效果, 另一方面也是做个备份.(具体出处我真忘了, 如果有人找到了源地址链接, 可以联系我加上)

```
#import "GLProgram.h"

// Base classes
#import "GPUImageOpenGLESContext.h"
#import "GPUImageOutput.h"
#import "GPUImageView.h"
#import "GPUImageVideoCamera.h"
#import "GPUImageStillCamera.h"
#import "GPUImageMovie.h"
#import "GPUImagePicture.h"
#import "GPUImageRawDataInput.h"
#import "GPUImageRawDataOutput.h"
#import "GPUImageMovieWriter.h"
#import "GPUImageFilterPipeline.h"
#import "GPUImageTextureOutput.h"
#import "GPUImageFilterGroup.h"
#import "GPUImageTextureInput.h"
#import "GPUImageUIElement.h"
#import "GPUImageBuffer.h"

// Filters
#import "GPUImageFilter.h"
#import "GPUImageTwoInputFilter.h"


#pragma mark - 调整颜色 Handle Color

#import "GPUImageBrightnessFilter.h"                //亮度
#import "GPUImageExposureFilter.h"                  //曝光
#import "GPUImageContrastFilter.h"                  //对比度
#import "GPUImageSaturationFilter.h"                //饱和度
#import "GPUImageGammaFilter.h"                     //伽马线
#import "GPUImageColorInvertFilter.h"               //反色
#import "GPUImageSepiaFilter.h"                     //褐色（怀旧）
#import "GPUImageLevelsFilter.h"                    //色阶
#import "GPUImageGrayscaleFilter.h"                 //灰度
#import "GPUImageHistogramFilter.h"                 //色彩直方图，显示在图片上
#import "GPUImageHistogramGenerator.h"              //色彩直方图
#import "GPUImageRGBFilter.h"                       //RGB
#import "GPUImageToneCurveFilter.h"                 //色调曲线
#import "GPUImageMonochromeFilter.h"                //单色
#import "GPUImageOpacityFilter.h"                   //不透明度
#import "GPUImageHighlightShadowFilter.h"           //提亮阴影
#import "GPUImageFalseColorFilter.h"                //色彩替换（替换亮部和暗部色彩）
#import "GPUImageHueFilter.h"                       //色度
#import "GPUImageChromaKeyFilter.h"                 //色度键
#import "GPUImageWhiteBalanceFilter.h"              //白平横
#import "GPUImageAverageColor.h"                    //像素平均色值
#import "GPUImageSolidColorGenerator.h"             //纯色
#import "GPUImageLuminosity.h"                      //亮度平均
#import "GPUImageAverageLuminanceThresholdFilter.h" //像素色值亮度平均，图像黑白（有类似漫画效果）

#import "GPUImageLookupFilter.h"                    //lookup 色彩调整
#import "GPUImageAmatorkaFilter.h"                  //Amatorka lookup
#import "GPUImageMissEtikateFilter.h"               //MissEtikate lookup
#import "GPUImageSoftEleganceFilter.h"              //SoftElegance lookup




#pragma mark - 图像处理 Handle Image

#import "GPUImageCrosshairGenerator.h"              //十字
#import "GPUImageLineGenerator.h"                   //线条

#import "GPUImageTransformFilter.h"                 //形状变化
#import "GPUImageCropFilter.h"                      //剪裁
#import "GPUImageSharpenFilter.h"                   //锐化
#import "GPUImageUnsharpMaskFilter.h"               //反遮罩锐化

#import "GPUImageFastBlurFilter.h"                  //模糊
#import "GPUImageGaussianBlurFilter.h"              //高斯模糊
#import "GPUImageGaussianSelectiveBlurFilter.h"     //高斯模糊，选择部分清晰
#import "GPUImageBoxBlurFilter.h"                   //盒状模糊
#import "GPUImageTiltShiftFilter.h"                 //条纹模糊，中间清晰，上下两端模糊
#import "GPUImageMedianFilter.h"                    //中间值，有种稍微模糊边缘的效果
#import "GPUImageBilateralFilter.h"                 //双边模糊
#import "GPUImageErosionFilter.h"                   //侵蚀边缘模糊，变黑白
#import "GPUImageRGBErosionFilter.h"                //RGB侵蚀边缘模糊，有色彩
#import "GPUImageDilationFilter.h"                  //扩展边缘模糊，变黑白
#import "GPUImageRGBDilationFilter.h"               //RGB扩展边缘模糊，有色彩
#import "GPUImageOpeningFilter.h"                   //黑白色调模糊
#import "GPUImageRGBOpeningFilter.h"                //彩色模糊
#import "GPUImageClosingFilter.h"                   //黑白色调模糊，暗色会被提亮
#import "GPUImageRGBClosingFilter.h"                //彩色模糊，暗色会被提亮
#import "GPUImageLanczosResamplingFilter.h"         //Lanczos重取样，模糊效果
#import "GPUImageNonMaximumSuppressionFilter.h"     //非最大抑制，只显示亮度最高的像素，其他为黑
#import "GPUImageThresholdedNonMaximumSuppressionFilter.h" //与上相比，像素丢失更多

#import "GPUImageSobelEdgeDetectionFilter.h"        //Sobel边缘检测算法(白边，黑内容，有点漫画的反色效果)
#import "GPUImageCannyEdgeDetectionFilter.h"        //Canny边缘检测算法（比上更强烈的黑白对比度）
#import "GPUImageThresholdEdgeDetectionFilter.h"    //阈值边缘检测（效果与上差别不大）
#import "GPUImagePrewittEdgeDetectionFilter.h"      //普瑞维特(Prewitt)边缘检测(效果与Sobel差不多，貌似更平滑)
#import "GPUImageXYDerivativeFilter.h"              //XYDerivative边缘检测，画面以蓝色为主，绿色为边缘，带彩色
#import "GPUImageHarrisCornerDetectionFilter.h"     //Harris角点检测，会有绿色小十字显示在图片角点处
#import "GPUImageNobleCornerDetectionFilter.h"      //Noble角点检测，检测点更多
#import "GPUImageShiTomasiFeatureDetectionFilter.h" //ShiTomasi角点检测，与上差别不大
#import "GPUImageMotionDetector.h"                  //动作检测
#import "GPUImageHoughTransformLineDetector.h"      //线条检测
#import "GPUImageParallelCoordinateLineTransformFilter.h" //平行线检测

#import "GPUImageLocalBinaryPatternFilter.h"        //图像黑白化，并有大量噪点

#import "GPUImageLowPassFilter.h"                   //用于图像加亮
#import "GPUImageHighPassFilter.h"                  //图像低于某值时显示为黑


#pragma mark - 视觉效果 Visual Effect

#import "GPUImageSketchFilter.h"                    //素描
#import "GPUImageThresholdSketchFilter.h"           //阀值素描，形成有噪点的素描
#import "GPUImageToonFilter.h"                      //卡通效果（黑色粗线描边）
#import "GPUImageSmoothToonFilter.h"                //相比上面的效果更细腻，上面是粗旷的画风
#import "GPUImageKuwaharaFilter.h"                  //桑原(Kuwahara)滤波,水粉画的模糊效果；处理时间比较长，慎用

#import "GPUImageMosaicFilter.h"                    //黑白马赛克
#import "GPUImagePixellateFilter.h"                 //像素化
#import "GPUImagePolarPixellateFilter.h"            //同心圆像素化
#import "GPUImageCrosshatchFilter.h"                //交叉线阴影，形成黑白网状画面
#import "GPUImageColorPackingFilter.h"              //色彩丢失，模糊（类似监控摄像效果）

#import "GPUImageVignetteFilter.h"                  //晕影，形成黑色圆形边缘，突出中间图像的效果
#import "GPUImageSwirlFilter.h"                     //漩涡，中间形成卷曲的画面
#import "GPUImageBulgeDistortionFilter.h"           //凸起失真，鱼眼效果
#import "GPUImagePinchDistortionFilter.h"           //收缩失真，凹面镜
#import "GPUImageStretchDistortionFilter.h"         //伸展失真，哈哈镜
#import "GPUImageGlassSphereFilter.h"               //水晶球效果
#import "GPUImageSphereRefractionFilter.h"          //球形折射，图形倒立

#import "GPUImagePosterizeFilter.h"                 //色调分离，形成噪点效果
#import "GPUImageCGAColorspaceFilter.h"             //CGA色彩滤镜，形成黑、浅蓝、紫色块的画面
#import "GPUImagePerlinNoiseFilter.h"               //柏林噪点，花边噪点
#import "GPUImage3x3ConvolutionFilter.h"            //3x3卷积，高亮大色块变黑，加亮边缘、线条等
#import "GPUImageEmbossFilter.h"                    //浮雕效果，带有点3d的感觉
#import "GPUImagePolkaDotFilter.h"                  //像素圆点花样
#import "GPUImageHalftoneFilter.h"                  //点染,图像黑白化，由黑点构成原图的大致图形


#pragma mark - 混合模式 Blend

#import "GPUImageMultiplyBlendFilter.h"             //通常用于创建阴影和深度效果
#import "GPUImageNormalBlendFilter.h"               //正常
#import "GPUImageAlphaBlendFilter.h"                //透明混合,通常用于在背景上应用前景的透明度
#import "GPUImageDissolveBlendFilter.h"             //溶解
#import "GPUImageOverlayBlendFilter.h"              //叠加,通常用于创建阴影效果
#import "GPUImageDarkenBlendFilter.h"               //加深混合,通常用于重叠类型
#import "GPUImageLightenBlendFilter.h"              //减淡混合,通常用于重叠类型
#import "GPUImageSourceOverBlendFilter.h"           //源混合
#import "GPUImageColorBurnBlendFilter.h"            //色彩加深混合
#import "GPUImageColorDodgeBlendFilter.h"           //色彩减淡混合
#import "GPUImageScreenBlendFilter.h"               //屏幕包裹,通常用于创建亮点和镜头眩光
#import "GPUImageExclusionBlendFilter.h"            //排除混合
#import "GPUImageDifferenceBlendFilter.h"           //差异混合,通常用于创建更多变动的颜色
#import "GPUImageSubtractBlendFilter.h"             //差值混合,通常用于创建两个图像之间的动画变暗模糊效果
#import "GPUImageHardLightBlendFilter.h"            //强光混合,通常用于创建阴影效果
#import "GPUImageSoftLightBlendFilter.h"            //柔光混合
#import "GPUImageChromaKeyBlendFilter.h"            //色度键混合
#import "GPUImageMaskFilter.h"                      //遮罩混合
#import "GPUImageHazeFilter.h"                      //朦胧加暗
#import "GPUImageLuminanceThresholdFilter.h"        //亮度阈
#import "GPUImageAdaptiveThresholdFilter.h"         //自适应阈值
#import "GPUImageAddBlendFilter.h"                  //通常用于创建两个图像之间的动画变亮模糊效果
#import "GPUImageDivideBlendFilter.h"               //通常用于创建两个图像之间的动画变暗模糊效果


#pragma mark - 尚不清楚
#import "GPUImageJFAVoroniFilter.h"
#import "GPUImageVoroniConsumerFilter.h"
tip8: H264硬编码
如果使用ijkplayer 使用硬解码, 一句代码即可.

// 开启硬解码
[option setPlayerOptionValue:@"1" forKey:@"videotoolbox"];

```
硬编码的应用场景: 我们要将主播的视频数据传送给服务器

通过摄像头来采集图像，然后将采集到的图像，通过硬编码的方式进行编码，最后编码后的数据将其组合成H264的码流通过网络传播。
摄像头采集图像, iOS系统提供了AVCaptureSession来采集摄像头的图像数据. 项目中我是直接使用 GPUImage 中的GPUImageVideoCamera, 直接设置GPUImageVideoCamera的代理即可, 在其代理方法- (void)willOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer;进行数据编码即可.

切记一点: 不管是系统自带的AVCaptureSession还是GPUImageVideoCamera采集到的数据都是未经过编码的CMSampleBuffer.
然后将采集到的数据, 用iOS开放的VideoToolbox进行硬编码. 关于VideoToolbox硬编解码网上很多教程, 当然最好是看Apple的官方文档, 如果只是硬编码, 看我的项目即可.

关键的编码函数(来自YOLO直播负责人的开源项目 BeautifyFaceDemo )

```

void didCompressH264(void *outputCallbackRefCon, void *sourceFrameRefCon, OSStatus status, VTEncodeInfoFlags infoFlags,
                     CMSampleBufferRef sampleBuffer )
{
    if (status != 0) return;
    // 采集的未编码数据是否准备好
    if (!CMSampleBufferDataIsReady(sampleBuffer))
    {
        NSLog(@"didCompressH264 data is not ready ");
        return;
    }
    ALinH264Encoder* encoder = (__bridge ALinH264Encoder*)outputCallbackRefCon;
    
    bool keyframe = !CFDictionaryContainsKey((CFArrayGetValueAtIndex(CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true), 0)), kCMSampleAttachmentKey_NotSync);
    
    if (keyframe) // 关键帧
    {
        CMFormatDescriptionRef format = CMSampleBufferGetFormatDescription(sampleBuffer);
        size_t sparameterSetSize, sparameterSetCount;
        const uint8_t *sparameterSet;
        OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 0, &sparameterSet, &sparameterSetSize, &sparameterSetCount, 0 );
        if (statusCode == noErr)
        {
            size_t pparameterSetSize, pparameterSetCount;
            const uint8_t *pparameterSet;
            OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 1, &pparameterSet, &pparameterSetSize, &pparameterSetCount, 0 );
            if (statusCode == noErr)
            {
                encoder->sps = [NSData dataWithBytes:sparameterSet length:sparameterSetSize];
                encoder->pps = [NSData dataWithBytes:pparameterSet length:pparameterSetSize];
                NSLog(@"sps:%@ , pps:%@", encoder->sps, encoder->pps);
            }
        }
    }
    
    CMBlockBufferRef dataBuffer = CMSampleBufferGetDataBuffer(sampleBuffer);
    size_t length, totalLength;
    char *dataPointer;
    OSStatus statusCodeRet = CMBlockBufferGetDataPointer(dataBuffer, 0, &length, &totalLength, &dataPointer);
    if (statusCodeRet == noErr) {
    
        size_t bufferOffset = 0;
        static const int AVCCHeaderLength = 4;
        while (bufferOffset < totalLength - AVCCHeaderLength)
        {
            uint32_t NALUnitLength = 0;
            memcpy(&NALUnitLength, dataPointer + bufferOffset, AVCCHeaderLength);
            NALUnitLength = CFSwapInt32BigToHost(NALUnitLength);
            NSData *data = [[NSData alloc] initWithBytes:(dataPointer + bufferOffset + AVCCHeaderLength) length:NALUnitLength];
            bufferOffset += AVCCHeaderLength + NALUnitLength;
            NSLog(@"sendData-->> %@ %lu", data, bufferOffset);
        }
    
    }

```



