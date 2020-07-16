---
layout: post
title: "Unity URP RenderPipeline"
featured-img: shane-rounce-205187
categories: [unity, render-pipeline]
---
# 简介

​	通用渲染管道(URP)是一个预先构建的脚本渲染管道，由Unity制作。URP提供了对艺术家友好的工作流，让您快速、轻松地跨一系列平台创建优化的图形，从移动设备到高端控制台和pc。

<br/><br/><br/>

# 需求（Requirement）

### unity editor兼容性

| Package version | Minimum Unity version | Maximum Unity version |
| :-------------: | :-------------------: | :-------------------: |
|     10.x.x      |      2020.2.0a17      |       2020.2.x        |
|      9.x.x      |      2020.1.0b6       |       2020.2.x        |
|      8.2.x      |      2020.1.0b6       |       2020.1.x        |
|      8.1.x      |      2020.1.0b6       |       2020.1.x        |
|      8.0.x      |      2020.1.0a23      |       2020.1.x        |
|      7.4.x      |      2019.3.2f1       |       2019.4.x        |
|      7.3.x      |      2019.3.2f1       |       2019.4.x        |
|      7.2.x      |      2019.3.0f6       |       2019.4.x        |
|      7.1.8      |      2019.3.0f3       |       2019.4.x        |

<br/>

### unity pipeline兼容性

​	使用URP制作的项目与高清渲染管道(HDRP)或内置的渲染管道不兼容。在开始开发之前，必须决定在项目中使用哪个渲染管道。有关选择渲染管道的信息，请参见Unity手册中的渲染管道部分。

<br/><br/><br/>

# 功能对比（Feature）

​	下表提供了URP渲染管线和内置管线间的功能概览对比。

​	**提示：**如果一个功能是In research，表示这个功能还在开发，如果是not supported，表示这个功能在任何版本都不计划被支持。

|                           Feature                            |    Built-in Render Pipeline Unity 2019.3    |                  Universal Render Pipeline                   |
| :----------------------------------------------------------: | :-----------------------------------------: | :----------------------------------------------------------: |
|                        ***Camera\***                         |                                             |                                                              |
|                        HDR rendering                         |                     Yes                     |                             Yes                              |
|                          HDR output                          |                     Yes                     |                         In research                          |
|                             MSAA                             |                     Yes                     |                             Yes                              |
|                       Physical Camera                        |                     Yes                     |                             Yes                              |
|                      Dynamic Resolution                      |                     Yes                     |                             Yes                              |
|                        Multi Display                         |                     Yes                     |                             Yes                              |
|                           Stacking                           |                     Yes                     |                             Yes                              |
|                         Flare Layer                          |                     Yes                     |                        Not supported                         |
|                        Depth Texture                         |                     Yes                     |                             Yes                              |
|                   Depth + Normals Texture                    |                     Yes                     |                        Not supported                         |
|                        Color Texture                         |                Not supported                |                             Yes                              |
|                        Motion vectors                        |                     Yes                     |                         In research                          |
|                       ***Batching\***                        |                                             |                                                              |
|                 Static Batching (By Shader)                  |                Not supported                |                             Yes                              |
|                Static Batching (By Material)                 |                     Yes                     |                             Yes                              |
|                       Dynamic Batching                       |                     Yes                     |                             Yes                              |
|                  Dynamic Batching (Shadows)                  |                     Yes                     |                         In research                          |
|                        GPU Instancing                        |                     Yes                     |                             Yes                              |
|                      ***Color Space\***                      |                                             |                                                              |
|                            Linear                            |                     Yes                     |                             Yes                              |
|                            Gamma                             |                     Yes                     |                             Yes                              |
|                    ***Realtime Lights\***                    |                                             |                                                              |
|          *Light Types* Directional Spot Point Area           |        Yes Yes Yes Rectangle (Baked)        |                Yes Yes Yes Rectangle (Baked)                 |
|                       Inner Spot Angle                       |                Not supported                |                             Yes                              |
|                           Shading                            |               Multiple Passes               |                         Single Pass                          |
|                *Culling* Per-Object Per-Layer                |                   Yes Yes                   |                           Yes Yes                            |
| *Light Limits* Main Directional Light Per Object  Per Camera |           1 Unlimited  Unlimited            | 1 8 (4 for GLES2). Can be point, spot, and directional Lights. 256 (16 for GLES 3.0 or lower, 32 on other mobile platforms) |
|                         Attenuation                          |                   Legacy                    |                        InverseSquared                        |
|                        Vertex Lights                         |                     Yes                     |                             Yes                              |
|                          SH Lights                           |                     Yes                     |                         In research                          |
|                   ***Realtime Shadows\***                    |                                             |                                                              |
|          *Light Types* Directional Spot Point Area           |          Yes Yes Yes Not supported          |          Yes - only 1 Yes In research Not supported          |
|           *Shadow Projection* Stable Fit Close Fit           |                   Yes Yes                   |                       Yes In research                        |
| *Shadow Cascades* Number of Cascades Control by Percentage Control by Distance |         1, 2 or 4 Yes Not supported         |                  1, 2 or 4 Yes In research                   |
|    *Shadow Resolve Type* Lighting Pass Screen Space Pass     |                   Yes Yes                   |                           Yes Yes                            |
|                         Shadow Bias                          |  Constant clip space offset + normal bias   | Offsets shadowmap texels in the light direction + normal bias |
|                     ***Lightmapping\***                      |                                             |                                                              |
|                          Enlighten                           |                     Yes                     |                        Not supported                         |
|                 Progressive Lightmapper, CPU                 |                     Yes                     |                             Yes                              |
|                 Progressive Lightmapper, GPU                 |                     Yes                     |                             Yes                              |
|             ***Realtime Global Illumination\***              |                                             |                                                              |
|                          Enlighten                           |                     Yes                     |                        Not supported                         |
|                  ***Mixed Lighting Mode\***                  |                                             |                                                              |
|                         Subtractive                          |                     Yes                     |                             Yes                              |
|                        Baked Indirect                        |                     Yes                     |                             Yes                              |
|                         Shadow Mask                          |                     Yes                     |                         In research                          |
|                     Distance Shadow Mask                     |                     Yes                     |                         In research                          |
|                     ***Light Probes\***                      |                                             |                                                              |
|                           Blending                           |                     Yes                     |                             Yes                              |
|                     Proxy Volume (LPPV)                      |                     Yes                     |                        Not supported                         |
|                       Custom Provided                        |                     Yes                     |                             Yes                              |
|                       Occlusion Probes                       |                     Yes                     |                             Yes                              |
|                   ***Reflection Probes\***                   |                                             |                                                              |
|                           Realtime                           |                     Yes                     |                             Yes                              |
|                            Baked                             |                     Yes                     |                             Yes                              |
|    *Sampling* Simple Blend Probes Blend Probes and Skybox    |                 Yes Yes Yes                 |                 Yes In research In research                  |
|                        Box Projection                        |                     Yes                     |                         In research                          |
|                    ***Lightmap Modes\***                     |                                             |                                                              |
|                       Non-Directional                        |                     Yes                     |                             Yes                              |
|                         Directional                          |                     Yes                     |                             Yes                              |
|                ***Environmental lighting\***                 |                                             |                                                              |
|                *Source* Skybox Gradient Color                |                 Yes Yes Yes                 |                         Yes Yes Yes                          |
|                *Ambient Mode* Realtime Baked                 |                   Yes Yes                   |                       In research Yes                        |
|                        ***Skybox\***                         |                                             |                                                              |
|                          Procedural                          |                     Yes                     |                             Yes                              |
|                           6 Sided                            |                     Yes                     |                             Yes                              |
|                           Cubemap                            |                     Yes                     |                             Yes                              |
|                          Panoramic                           |                     Yes                     |                             Yes                              |
|                          ***Fog\***                          |                                             |                                                              |
|                            Linear                            |                     Yes                     |                             Yes                              |
|                         Exponential                          |                     Yes                     |                             Yes                              |
|                     Exponential Squared                      |                     Yes                     |                             Yes                              |
|               ***Visual Effects Components\***               |                                             |                                                              |
|                             Halo                             |                     Yes                     |                        Not supported                         |
|                          Lens Flare                          |                     Yes                     |                        Not supported                         |
|                        Trail Renderer                        |                     Yes                     |                             Yes                              |
|                      Billboard Renderer                      |                     Yes                     |                             Yes                              |
|                          Projector                           |                     Yes                     |                        Not supported                         |
|                   ***Shaders (General)\***                   |                                             |                                                              |
|                         Shader Graph                         |                Not supported                |                             Yes                              |
|                       Surface Shaders                        |                     Yes                     |                        Not supported                         |
|                  Camera-relative Rendering                   |                Not supported                |                         In research                          |
| *Built-in Lit Uber Shader* Metallic Workflow Specular Workflow |           Standard Shader Yes Yes           | [Lit Shader](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/lit-shader.html) Yes Yes |
| *Surface Type and Blend Mode* Opaque Faded (Alpha Blend) Transparent Cutout Additive Multiply | Yes Yes Yes Yes Not supported Not supported |                   Yes Yes Yes Yes Yes Yes                    |
| *Surface Inputs* Albedo (Base Map) Specular Metallic Smoothness Ambient Occlusion Normal Map Detail Map Detail Normal Map Heightmap |     Yes Yes Yes Yes Yes Yes Yes Yes Yes     | Yes Yes Yes Yes Yes Yes Not supported Not supported Not supported |
|                        Light Cookies                         |                     Yes                     |                         In research                          |
|                       Parallax Mapping                       |                     Yes                     |                        Not supported                         |
|                     Light Distance Fade                      |                Not supported                |                         In research                          |
|                     Shadow Distance Fade                     |                     Yes                     |                         In research                          |
|                   Shadow Cascade Blending                    |                Not supported                |                         In research                          |
|                        GPU Instancing                        |                     Yes                     |                             Yes                              |
|                       Double Sided GI                        |                     Yes                     |                             Yes                              |
|                          Two Sided                           |                Not supported                |                             Yes                              |
|                        Order In Layer                        |                Not supported                |                             Yes                              |
|                 ***Render Pipeline Hooks\***                 |                                             |                                                              |
|                   Camera.RenderWithShader                    |                     Yes                     |                        Not supported                         |
| Camera.AddCommandBuffer* (Camera.Remove[All]CommandBuffer*)  |                     Yes                     |                        Not supported                         |
|                        Camera.Render                         |                     Yes                     |                        Not supported                         |
|   Light.AddCommandBuffer* (LightRemove[All]CommandBuffer*)   |                     Yes                     |                        Not supported                         |
|                          OnPreCull                           |                     Yes                     |                        Not supported                         |
|                         OnPreRender                          |                     Yes                     |                        Not supported                         |
|                         OnPostRender                         |                     Yes                     |                        Not supported                         |
|                        OnRenderImage                         |                     Yes                     |                        Not supported                         |
|                        OnRenderObject                        |                     Yes                     |                             Yes                              |
|                      OnWillRenderObject                      |                     Yes                     |                             Yes                              |
|                       OnBecameVisible                        |                     Yes                     |                             Yes                              |
|                      OnBecameInvisible                       |                     Yes                     |                             Yes                              |
|                 Camera Replacement Material                  |                Not supported                |                         In research                          |
|              RenderPipeline.BeginFrameRendering              |                Not supported                |                             Yes                              |
|               RenderPipeline.EndFrameRendering               |                Not supported                |                             Yes                              |
|             RenderPipeline.BeginCameraRendering              |                Not supported                |                             Yes                              |
|              RenderPIpeline.EndCameraRendering               |                Not supported                |                             Yes                              |
|          UniversalRenderPipeline.RenderSingleCamera          |                Not supported                |                             Yes                              |
|                     ScriptableRenderPass                     |                Not supported                |                             Yes                              |
|                       Custom Renderers                       |                Not supported                |                             Yes                              |
|                    ***Post-processing\***                    |   Uses Post-Processing Version 2 package    | Uses integrated [post-processing solution](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/integration-with-post-processing.html) |
|                   Ambient Occlusion (MSVO)                   |                     Yes                     |                         In research                          |
|                        Auto Exposure                         |                     Yes                     |                        Not supported                         |
|                            Bloom                             |                     Yes                     |                             Yes                              |
|                     Chromatic Aberration                     |                     Yes                     |                             Yes                              |
|                        Color Grading                         |                     Yes                     |                             Yes                              |
|                        Depth of Field                        |                     Yes                     |                             Yes                              |
|                            Grain                             |                     Yes                     |                             Yes                              |
|                       Lens Distortion                        |                     Yes                     |                             Yes                              |
|                 *Motion Blur* Camera Object                  |              Yes Not supported              |                       Yes In research                        |
|                   Screen Space Reflections                   |                     Yes                     |                        Not supported                         |
|                           Vignette                           |                     Yes                     |                             Yes                              |
|                       ***Particles\***                       |                                             |                                                              |
|                       VFX Graph (GPU)                        |                Not supported                |                             Yes                              |
|                    Particles System (CPU)                    |                     Yes                     |                             Yes                              |
| *Shaders* Physically Based Simple Lighting (Blinn Phong) Unlit |                 Yes Yes Yes                 | Yes ([Particles Lit](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/particles-lit-shader.html)) Yes ([Particles Simple Lit](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/particles-simple-lit-shader.html)) Yes ([Particles Unlit](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/particles-unlit-shader.html)) |
|                        Soft Particles                        |                     Yes                     |                             Yes                              |
|                          Distortion                          |                     Yes                     |                             Yes                              |
|                      Flipbook Blending                       |                     Yes                     |                             Yes                              |
|                        ***Terrain\***                        |                                             |                                                              |
| *Shaders* Physically Based Simple Lighting (Blinn-Phong) Unlit Speed Tree Vegetation Detail |      Yes Yes Not supported Yes Yes Yes      |           Yes In research In research Yes Yes Yes            |
|                          Wind Zone                           |                     Yes                     |                             Yes                              |
|                       Number of Layers                       |                  Unlimited                  |                              8                               |
|                     GPU Patch Generation                     |                     Yes                     |                             Yes                              |
|                         Surface Mask                         |                Not supported                |                         In research                          |
|                          ***2D\***                           |                                             |                                                              |
|                            Sprite                            |                     Yes                     |                             Yes                              |
|                           Tilemap                            |                     Yes                     |                             Yes                              |
|                         Sprite Shape                         |                     Yes                     |                             Yes                              |
|                        Pixel-Perfect                         |  Yes - using the 2D Pixel Perfect Package   |                             Yes                              |
|                          2D Lights                           |                Not supported                |                             Yes                              |
|                 ***UI (Canvas Renderer)\***                  |                                             |                                                              |
|                    Screen Space - Overlay                    |                     Yes                     |                             Yes                              |
|                    Screen Space - Camera                     |                     Yes                     |                             Yes                              |
|                         World Space                          |                     Yes                     |                             Yes                              |
|                        Text Mesh Pro                         |                     Yes                     |                             Yes                              |
|                          ***VR\***                           |                                             |                                                              |
|                          Multipass                           |                     Yes                     |                             Yes                              |
|                         Single Pass                          |                     Yes                     |                             Yes                              |
|                    Single Pass Instanced                     |                     Yes                     |                             Yes                              |
| *Post-processing* Oculus Rift Oculus Quest Oculus Go Gear VR PSVR HoloLens WMR Magic Leap One OpenVR |     Yes Yes Yes Yes Yes Yes Yes Yes Yes     |              Yes Yes Yes Yes Yes Yes Yes Yes No              |
|                          ***AR\***                           |                                             |                                                              |
|                        AR Foundation                         |                     No                      |                             Yes                              |
|                         ***Debug\***                         |                                             |                                                              |
|                       Scene view modes                       |                     Yes                     |                         In research                          |

<br/>

<br/>

<br/>

# 开始使用URP（Getting Started）

### 在一个新工程中使用URP

如果想要在一个新工程中使用URP渲染管线，可以通过模板创建一个URP新工程。

步骤如下：

1. 打开unityhub。
2. 在主页点击新工程，然后在弹出的模板选择URP模板。
3. 点击创建。然后unity就会为你创建一个新工程，在这个工程中，URP会被安装和配置，并且会包括一些示例内容用于演示URP的功能。
4. 在Project视窗，找到Assets文件夹，点击ReadMe assest，unity会在Inspector视窗中显示工程信息。

<br/>

### 在已经存在的项目中添加URP

​	要在一个已经存在的项目中使用URP，你可以通过PackageManager(Window->PackageManager)安装最新的URP包到项目中。

<br/>

​	URP渲染管线集成了自己的**后处理解决方法**。如果你在给项目升级成URP的时候，旧项目中安装了Post Processing version 2 package的话，你需要将项目中的此后处理包给删除掉。当你在安装了URP之后，你可以重新实现旧项目中的后处理效果。

​	**注意：**URP渲染管线目前**不支持客制后处理效果**，如果你的旧项目中包含了客制后处理效果，这些客制化效果无法被重新实现，客制化后处理效果会在URP将来的版本中支持。

<br/>

<br/>

### 安装URP

1. 在unity中，打开想要升级成URP的项目。
2. window->PackageManager来打开PackageManager窗口。
3. 选择All选项，此选项显示当前运行的unity版本的可用包列表。
4. 在PackageManager窗口中选择Universal RP。
5. 在PackageManager窗口中的右下角，点击Install按钮对URP进行安。

<br/>

### 配置URP

​	在你使用URP渲染管线之前，还需要配置URP，你需要创建一个可编程渲染管线资源并且调整项目中的Graphics设置。

#### 一、创建Universal RenderPipeline Assest

​	**Universal RenderPipeline Assest**控制着项目中的全局渲染和质量设置，并且创建一个render Pipeline实例，这个实例包含了中间资源和Render Pipeline的实现。

创建一个Universal RenderPipeline Assest：

1. 在Project窗口中
2. 点击右键并且选择Create->Rendering->Universal Render Pipeline->Pipeline Assest。或者在顶部菜单中Assests->Create->Rendering->Universal Render Pipeline->Pipeline Assest创建一个Universal Render Pipeline Assest资源。

你可以为这个资源重命名，也可以保留默认名称。

<br/>

##### 二、将创建的资源添加到Graphic设置中

​	为了使用URP，需要将新创建的Universal Render Pipeline Assest添加到Graphic设置中。如果不这样做的话，unity会继续尝试使用内置渲染管线。

1. **Edit->Project Settings...->Graphic**。
2. 在Scriptable Render Pipeline Settings区域，加上新创建的Universal Render Pipeline Assest。当你添加了此资源之后，Graphic设置会马上更改，所以你的项目现在就是已经在使用URP渲染管线了。

<br/>

<br/>

<br/>

### Universal Render Pipeline Asset

​	URP资产控制通用渲染管道的几个图形特性和质量设置。它是一个可编写脚本的对象，继承自“RenderPipelineAsset”。当你在图形设置中设置资产时，Unity会从内置的渲染管道切换到URP。然后，您可以直接在URP中调整相应的设置，而不是在其他地方寻找它们。

​	您可以拥有多个URP资产并在它们之间进行切换。例如，你可以设置一个打开阴影和一个关闭阴影。如果你切换属性来查看效果，你不必每次都手动切换相应的阴影设置。但是，由于渲染管道不兼容，您不能在HDRP/SRP和URP资产之间切换。

​	在URP中，你可以配置以下设置：

- [**General**](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html#general)
- [**Quality**](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html#quality)
- [**Lighting**](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html#lighting)
- [**Shadows**](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html#shadows)
- [**Post-processing**](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html#post-processing)
- [**Advanced**](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html#advanced)
- Adaptive Performance

**注意：**如果你在（menu：Graphic->Settings->add the 2D Render Assest under **Scriptable Render Pipeline Settings**）中开启了experimental 2D Renderer，那么URP资产中和3D相关的一些修改最终可能并不会影响你的最终程序或者游戏。

<br/>

#### 1、General

General的设置控制渲染管线 **渲染帧**的核心部分。

|      **Property**       | **Description**                                              |
| :---------------------: | :----------------------------------------------------------- |
|    **Depth Texture**    | 允许URP创建摄像机_CameraDepthTexture（摄像机深度纹理），然后此深度纹理作为场景中所有摄像使用的默认深度纹理。你可以从摄像机的Inspector窗口中为个别摄像机覆盖此项设置。 |
|   **Opaque Texture**    | 允许URP创建_CameraOpaqueTexture作为场景中所有摄像机的默认使用纹理 CameraOpaqueTexture纹理就像在内置渲染管线中使用GrabPass得到的效果一样。CameraOpaqueTexture纹理提供在URP渲染任何透明网格之前提供场景的快照。你可以在透明着色器中使用它来创建像磨砂玻璃，水折射，或者热波的效果。你可以从摄像机的Inspector窗口中为个别摄像机覆盖此项设置。 |
| **Opaque Downsampling** | 将不透明的纹理的采样模式设置为以下模式之一：<br />**None**：生成与相机具有相同分辨率的不透明通道的副本。<br />**2x Bilinear**：产生半分辨率图像与双线性滤波。<br />**4x Box**：产生一个四分之一分辨率的图像与框过滤。这会产生一个柔和模糊的副本。<br/>**4x Bilinear**：产生四分之一分辨率的图像与双线性滤波。 |
|    **Terrain Holes**    | 如果你禁用了这个选项，URP就会移除所有的地形洞（Terrain hole）着色器变体（Shader variants），这就减少了建造时间。 |

<br/>

#### 2、Quality

​	这些设置控制URP的质量水平。在这种情况下，您可以在低端硬件上提高性能，或者在高端硬件上使图形看起来更好。

​	**提示**:如果您想为不同的硬件设置不同的设置，您可以跨多个通用渲染管道资产配置这些设置，并根据需要切换它们。

| Property         | Description                                                  |
| :--------------- | :----------------------------------------------------------- |
| **HDR**          | 启用此功能可以默认场景中的每个摄像机都是使用高动态范围(HDR)渲染。使用HDR，图像最亮的部分可以大于1。这给你一个更大范围的光强度，所以你的照明看起来更真实。有了它，即使在明亮的光线下，你仍然可以看到细节，体验较少的饱和度。这是有用的，如果你想要广泛的照明或使用Bloom效果。如果您的目标是低端硬件，可以禁用此功能以跳过HDR计算并获得更好的性能。您可以在相机检查器中为单个相机覆盖此选项。 |
| **MSAA**         | 在渲染时，默认为场景中的每个相机使用多样本抗锯齿（[Multi Sample Anti-aliasing](https://en.wikipedia.org/wiki/Multisample_anti-aliasing)）进行渲染。这样可以软化几何图形的边缘，使它们不会锯齿状或闪烁。在下拉菜单中，选择每个像素使用多少个样本:**2x、4x或8x**。选择的样本越多，对象的边缘就越平滑。如果您想跳过MSAA计算，或者在2D游戏中不需要它们，请选择禁用。您可以在相机检查器中为单个相机覆盖此选项。 |
| **Render Scale** | 这个滑动条可以缩放渲染目标的分辨率(不是当前设备的分辨率)。当您出于性能原因想要以较小的分辨率进行渲染或为了提高渲染质量而使用此方法时。这只会缩放游戏渲染。UI呈现给设备的分辨率还是本来的分辨率。 |

<br/>

#### 3、Lighting

​	这些设置会影响场景中的灯光。

​	如果你禁用这些设置，相关的关键字将从着色器变量中剥离（Shader 中的一些变量不会被正确赋值，即一些变量的值会被剥离）。如果有一些你肯定不会在游戏或应用中使用的设置，你可以禁用它们来提高性能并减少构建时间。

| Property              | Description                                                  |
| :-------------------- | :----------------------------------------------------------- |
| **Main Light**        | 这些设置会影响场景中的**主 方向光**。您可以通过在照明检查器（Lighting Inspector）中将它指定为一个太阳光源来选择它。如果你没有指定一个太阳光源，URP会将场景中最亮的方向光作为主 光。你可以在逐像素照明和无照明之间选择。如果你选择None, URP不会渲染主光，即使你设置了一个太阳光源 |
| **Cast Shadows**      | 选中此选项，让主光源在场景中投射阴影。                       |
| **Shadow Resolution** | 这控制了主光源的阴影贴图纹理的大小。高分辨率可以产生更清晰、更详细的阴影。如果内存或渲染时间是一个问题，尝试低分辨率。 |
| **Additional Lights** | 通过此选项，你可以选择有额外的灯来补充你的主灯。选择 **逐顶点（Per Vertex）、逐像素（Per Pixel）、禁用（Disabled）**。 |
| **Per Object Limit**  | 这个滑动条设置了限制多少额外的灯光可以影响每个游戏对象。     |
| **Cast Shadows**      | 选择此项，让额外的灯光在你的场景中投射阴影。                 |
| **Shadow Resolution** | 这控制了纹理的大小，为额外的灯光投射方向阴影。这是一个精灵图集，包含了16个阴影贴图(shadow maps)。高分辨率可以产生更清晰、更详细的阴影。如果内存或渲染时间是一个问题，尝试低分辨率。 |

<br/>

#### 4、shadow

​	这些设置影响阴影的外观和行为。它们还会影响性能，所以你可以在这方面进行调整，以获得视觉质量和阴影渲染速度之间的最佳平衡。

| Property         | Description                                                  |
| :--------------- | :----------------------------------------------------------- |
| **Distance**     | **它控制了摄像机物体投射阴影的距离**(以Unity单位为准)。在这个距离之后，URP不渲染阴影。例如，值100意味着距离摄像机超过100米的物体不会产生阴影。在大的、开放的世界中使用它，在那里渲染远处的阴影会消耗大量内存。或者在视图距离有限的自上而下游戏中使用它。 |
| **Cascades**     | 为阴影选择级联(cascades)的数目。高数量的cascades给你更详细的阴影靠近相机。选项有:**None**,**Two Cascades**和**Four Cascades**。如果遇到性能问题，请尝试降低级联的数量。你也可以在设置下面的部分配置阴影的距离。离相机越远，阴影的细节就越少。 |
| **Soft Shadows** | 如果你已经为**主光(Main Light)**或**附加光(Additional Light)**启用了阴影，你可以在阴影贴图上添加一个平滑的过滤。这会使你的阴影边缘更加平滑。启用后，渲染管线在桌面平台上执行一个5x5的**Tent fliter**过滤器，在移动设备上执行一个4 Tap过滤器。当禁用时，渲染管线使用默认的硬件过滤对阴影进行一次采样。如果你禁用这个功能，你会得到更快的渲染，但是阴影边缘可能会更尖锐，也有可能像素化。 |

<br/>

#### 5、Post-processing

此项设置允许对处理设置进行微调。

| Property         | Description                                                  |
| :--------------- | :----------------------------------------------------------- |
| **Grading Mode** | 选择项目要使用的颜色分级模式:<br/>**High Dynamic Range**:这种模式最适合高精度分级，类似电影制作工作流程。统一在调色前应用颜色分级。<br/>**Low Dynamic Range**:此模式遵循更经典的工作流。统一在调色后应用有限范围的颜色分级。 |
| **LUT Size**     | 设置通用渲染管道用于颜色分级的内部和外部查找纹理[look-up textures (LUTs)](https://docs.unity3d.com/Manual/PostProcessing-ColorGrading.html)的大小。更大的尺寸提供了更精确的精度，但有性能和内存使用的潜在成本。你不能混合和匹配LUT大小，所以在你开始颜色分级过程之前决定一个大小。**默认值 32**提供了速度和质量的良好平衡。 |

<br/>

#### 6、Advanced

本节允许您微调不太常更改的设置，这些设置会影响更深层次的渲染特性和着色器组合。

| Property                     | Description                                                  |
| :--------------------------- | :----------------------------------------------------------- |
| **SRP Batcher**              | 选中此项以启用**SRP批处理程序（SRP Batcher）**。如果你有许多不同的材质使用相同的着色器的时候这将相当有用。SRP批处理程序是一个内部循环，加速CPU渲染而不影响GPU性能。当您使用SRP批处理程序时，它将替换SRP呈现代码的内部循环。 |
| **Dynamic Batching**         | 启用动态批处理（[Dynamic Batching](https://docs.unity3d.com/Manual/DrawCallBatching.html)），使渲染管道自动批处理共享同一材质的动态小对象。这对于不支持GPU实例化的平台和图形api非常有用。如果你的目标硬件确实支持GPU实例化，禁用**Dynamic Batching**。您可以在运行时更改这一点。 |
| **Mixed Lighting**           | 启用Mixed Lighting（Light Mode为Mixed的光源），告诉管道在构建中包含混合照明着色器变体。 |
| **Debug Level**              | 设置渲染管线生成的调试信息的级别。值为:<br/>**Disabled** : 禁用调试。这是默认值。<br/>**Profile** : 使呈现管道提供详细的信息标记，您可以在FrameDebugger中看到。 |
| **Shader Variant Log Level** | Set the level of information about Shader Stripping and Shader Variants you want to display when Unity finishes a build. Values are: **Disabled**: Unity doesn’t log anything. **Only Universal**: Unity logs information for all of the [URP Shaders](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/shaders-in-universalrp.html). **All**: Unity logs information for all Shaders in your build. You can see the information in Console panel when your build has finished.当Unity完成一个构建时，设置你想要显示的着色器剥离和着色器变体的信息级别。值为 :<br/>**Disable** : Unity不记录任何内容。<br/>**Only Universal** : Unity记录所有[URP Shaders](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/shaders-in-universalrp.html)的信息。<br/>**All** : Unity记录你构建中所有着色器的信息。当构建完成时，您可以在控制台面板中看到这些信息。 |

<br/>

#### 7、Adaptive Performance（自适应性能）

如果安装了Adaptive Performance包，则显示此部分。它允许改变设置如何适应性能和渲染管道交互。

| **Property**                 | **Description**                      |
| :--------------------------- | :----------------------------------- |
| **Use adaptive performance** | 允许自适应性能调整渲染质量在运行时。 |

<br/><br/><br/>

# 渲染（Rendering）

URP渲染管线使用下面的功能来渲染场景：

- 前向渲染（Forward renderer）
- URP内置的渲染模型（ [Shading models](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/shading-model.html) for shaders shipped with URP）
- 相机（Camera）
- URP资产（ [UniversalRP Asset](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-asset.html)）

在前向渲染中，URP渲染管线实现了告诉unity如何去渲染帧的渲染循环。

![](../assets/img/resources/URP_Rendering_Flowchart.png)

​	当渲染管道在图形设置中处于活动状态时（Universal Rendering Pipeline Assest在图形设置中使用的时候），Unity使用URP渲染在项目中的所有相机，包括game、Scence view camera、反射探针（Reflection Probes）以及Inspector的预览窗口。

​	URP渲染器为每个摄像头执行一个摄像头循环，执行以下步骤:

1. 筛选场景中渲染的对象
2. 为渲染器创建数据
3. 执行将图像输出到framebuffer的渲染器。



​	**URP提供了回调**，您可以使用它在渲染循环的开始和结束，以及在每个相机循环的开始和结束时执行代码。

<br/>

<br/>

### Camera loop

相机环执行以下步骤:

| Step                         | Description                                                  |
| :--------------------------- | :----------------------------------------------------------- |
| **Setup Culling Parameters** | 配置决定剔除系统光照（Lights）阴影（Shadows）的参数，您可以使用自定义渲染器（Render）覆盖渲染管线的这一部分。 |
| **Culling**                  | 根据上一步的剔除参数来计算一个对摄像机可见的  可见渲染器（Visible Render）、shadow Caster、Light的列表，剔除参数和摄像机图层距离（Layer Distance）影响剔除和渲染性能。 |
| **Build Rendering Data**     | 捕获基于上一步的Culling输出、URP Assest、摄像机和当前运行平台的质量设置的信息，以构建渲染数据。渲染数据告诉渲染器为当前相机和当前选择的平台的渲染工作 的数量和所需的质量。 |
| **Setup Renderer**           | 构建一个（Render Pass）列表，并根据渲染数据对它们进行队列执行。您可以使用自定义渲染器覆盖渲染管线的这一部分。 |
| **Execute Renderer**         | 执行队列中的每个渲染传递。渲染器将摄像机图像输出到framebuffer。 |



















































