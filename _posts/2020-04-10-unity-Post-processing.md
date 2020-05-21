---
layout: post
title: "Unity 后处理"
featured-img: shane-rounce-205187
categories: [unity, Post-processing]
---

## Post-processing

​	后处理应用全屏过滤器和效果到相机的图像缓冲区在图像出现在屏幕上之前，它可以极大地改善应用程序的视觉效果，而只需要很少的设置时间。您可以使用后期处理效果来模拟物理相机和胶片属性。





## Post-processing and render pipelines

后处理效果的使用取决于你使用的渲染管线，不同的渲染管线，使用后处理的方法也是不同的。

- Built-in RenderPipeline（内置渲染管线）
  - 首先得安装Post-processing Version 2 package，然后在摄像机上添加Post-process Layer组件具体参考[戳这里](https://docs.unity3d.com/Packages/com.unity.postprocessing@2.3/manual/Quick-start.html)。

- Universal RenderPipeline（URP渲染管线），使用Volumes系统使用后处理效果。
- High Definition Render Pipeline（HDRP高清渲染管线），使用Volumes系统使用后处理效果。



## Post-processing Effects

#### 1、Ambient Occlusion（环境光遮蔽）简介

环境光遮蔽后处理效果使得 相互接近的折痕、孔洞、交叉点和表面变暗。

![](../assets/img/resources/PostProcessing-AmbientOcclusion-0.jpg)





![](/home/android/桌面/GitTest/MonkeyGamerPan.github.io/assets/img/resources/PostProcessing-AmbientOcclusion-1.jpg)

​	前一张图片是没有使用环境光遮蔽后处理效果的，而后一张相反，可以看出后一张图片中的在墙角、孔洞、柱子底部等地方明显变得更暗。

#### 2、Ambient Occlusion（环境光遮蔽）使用

Ambient Occlusion环境光后处理效果一共有两种模式

- Scalable Ambient Obscurance
- Multi-scale Volumetric Occlusion



##### 	Scalable Ambient Obscurance	

​	**这个模式是在不支持 compute shader的平台上使用的**，这个是在旧平台上工作的Ambient Obscurance（环境光遮蔽）的一个标准实现，如果你需要以支持计算的平台为目标，那么可以使用 **Multi-scale Volumetric Occlusion** 模式。

​	这种模式可能会消耗大量资源，尤其是在非常靠近摄像机的情况下。为了提高性能，可以使用一个小半径设置，对距离源像素很近且位于剪切空间的像素进行采样。这使得缓存更有效。使用更大的半径设置会生成距离源像素更远的样本，并且不会从缓存中获益，这会降低效果。**建议使用较小的半径范围**。

​	由于相机的视角，靠近前平面的物体比远平面的物体使用更大的半径，因此计算靠近相机的物体的环境光遮蔽通过速度比远平面的物体要慢，远平面的物体在屏幕上只占几个像素。

​	取消质量设置或者降低质量设置也会提高性能

​	**Scalable Ambient Obscurance**不应该在移动平台或控制台上使用，因为**Multi-scale Volumetric Occlusion**模式更快，为这些平台提供更好的图形。

![](../assets/img/resources/ssao-1.png)



- Intensity : 调节Ambient Occlusion后处理效果产生的暗部的深浅

- Readius : 设置采样点的半径，控制暗区范围。
- Quality : 定义影响质量和性能的采样点的数量。
- Color : 设置环境光遮蔽的颜色。
- Ambient Only : 仅环境光启用此复选框，使环境光遮蔽效果仅影响环境光。此选项仅可用于延迟呈现路径和HDR呈现。



##### Multi-scale Volumetric Occlusion

**这个模式运行在支持Compute shader的设备上**，这个模式针对控制台和桌面平台进行了优化，拥有着比Scalable Ambient Obscurance更好的图形显示以及更快的运行速度。

![](../assets/img/resources/ssao-2.png)



- Thickness Modifier : 修改（occluders）咬合器的厚度。这增加了暗区，但会在物体周围引入暗晕。

其他参数和上面一致。



## Anti-aliasing（图形抗锯齿效果）

​	Anti-aliasing（图形抗锯齿）后处理效果会让图形拥有一个平滑的外观，抗锯齿算法是基于图像的，在不支持传统的多采样时非常有用，比如在Unity 5.5或更早版本中， [deferred rendering](https://docs.unity3d.com/Manual/RenderTech-DeferredShading.html) shading path（延迟渲染着色路径）或正向渲染路径中的HDR。编辑器的 [Quality settings](https://docs.unity3d.com/Manual/class-QualitySettings.html) （质量设置）窗口是这些选项的主页。

​	抗锯齿效果给图形一个平滑的外观。混叠是线条出现锯齿状或具有“楼梯”外观的一种效果(如下面的左图所示)。如果图形输出设备没有足够高的分辨率来显示直线，就会出现这种情况。

​	使用抗混叠减少这些锯齿线的突出，通过周围的中间色调的颜色。尽管这减少了线条的锯齿状外观，但也使它们更加模糊。

​	它们是在后期处理层组件中为每个相机设置的。

![](../assets/img/resources/PostProcessing-Antialiasing-0.jpg)





​	Anti-aliasing（图形抗锯齿）在后处理堆栈中可用的算法有：	

- **Fast Approximate Anti-aliasing (FXAA) **:  一个给不支持运动矢量的移动平台和其他平台的快速算法
- **Subpixel Morphological Anti-aliasing (SMAA)** : 一个高质量但速度较慢的算法，适用于不支持运动矢量的移动平台和其他平台。
- **Temporal Anti-aliasing (TAA) **: 一种需要运动矢量的先进技术。适用于桌面和控制台平台



##### Fast Approximate Anti-aliasing (FXAA)

​	FXAA是最有效的技术，推荐用于不支持运动矢量的移动平台和其他平台.

![](../assets/img/resources/aa-1.png)

- Fast Mode : 启用此复选框以获得质量较低但速度更快的FXAA变体。推荐用于移动平台。
- Keep Alpha : 如果需要保持alpha通道不受后处理影响，请启用此复选框。如果被禁用，Unity将使用alpha通道来存储用于加速和提高视觉质量的内部数据。



##### Subpixel Morphological Anti-aliasing (SMAA) 

​	SMAA具有比FXAA更高质量的抗锯齿效果，但它的速度也更慢。取决于你的游戏的艺术风格，它在避免了一些技术缺点之后可以像Temporal Anti-aliasing (TAA）一样工作。**不支持AR/VR。**

![](../assets/img/resources/aa-2.png)

- ​	Quality ：设置整体质量的抗混叠过滤器。

  降低质量设置会使效果运行得更快。不要在移动平台上使用SMAA。

  

##### Temporal Anti-aliasing (TAA) 

​	TAA是一种先进的抗锯齿技术，它在历史缓冲区中不断积累帧，以便更有效地平滑边缘。它在平滑运动中的边缘方面要比FXAA好得多，但它需要运动矢量，而且比FXAA更昂贵。它是桌面和控制台平台的理想选择。**不支持GLES2品台**。

![](../assets/img/resources/aa-3.png)

​	

- Jitter Spread ：设置散布抖动样本的直径(以texel为单位)。值越小，输出越清晰，但别名越多。更大的值会产生更稳定但更模糊的输出。
- Stationary Blending：设置固定碎片的混合系数。此设置控制历史样本混合到最终颜色中的百分比，以使活动最小。
- Motion Blending：设置移动碎片的混合系数。此设置控制历史样本混合到最终颜色中的比例，用于具有显著主动运动的碎片。
- Sharpness：设置锐度，以减轻TAA可能导致的高频区域细节的轻微丢失。





## Auto Exposure（自动曝光效果）

​	Auto Exposure（自动曝光效果）根据图像包含的亮度级别范围动态调整图像的曝光。

​	在Unity中，这种效果会在每一帧生成一个直方图，并对其进行过滤，以找到平均亮度值。只有支持compute shader 的品台才支持。这种效果模拟人眼根据实际亮度调整感知亮度，这个效果何在屏幕偏暗或者偏亮的时候自动调整画面曝光度，这个效果中的曝光度以EV单位。

![](../assets/img/resources/autoexposure.png)

​	

- Filtering：设置直方图的上下百分比，找到一个稳定的平均亮度。超出此范围的值将被丢弃，并且不会影响平均亮度。从当前画面的亮度直方图中选取一个范围，当前画面应该使用的曝光度是从这个范围的平均亮度计算出来的。一般来说要避免计算画面中的极亮或者极暗的部分，才能得到正确的曝光度。

- Minimum：设置最小平均亮度。

- Maximun：设置最大平均亮度

  两个都是曝光度范围，由上面计算得到的曝光度必须在这个范围内（默认是0所以画面不会有任何变化），比如某个场景中都是暗部，平均亮度太低，曝光度就会很高，得到的就是过亮的场景了。相反，不设置上限，场景都是较亮的部分，曝光就会太低。

- EXposure Compensation：设置中灰色值来补偿场景的全局曝光。曝光补偿，总的亮度调节。

- Type：选择适配类型。**Progressive**模式下，可以通过调整Speed up、down参数来分别确定曝光度的增减速度，会有一种过度效果。**Fixed**模式下则会直接应用计算得到的曝光度，在明暗突变的情况下，Progressive就像人的眼睛一样产生逐渐适应的效果，而Fixed则是直接突变。

- Speed up：设置从黑暗环境到光明环境的适应速度。

- Speed down：设置从光到暗环境的适应速度。

  



























































