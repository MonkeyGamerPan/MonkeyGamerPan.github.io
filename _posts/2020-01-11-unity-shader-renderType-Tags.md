---
layout: post
title: "Unity shader中的渲染状态和Tags"
featured-img: shane-rounce-205187
categories: [unity, address]
---







## 渲染状态

​	ShaderLab提供了一系列渲染状态的设置指令，这些指令可以设置显卡的各种状态，例如是否开启混合/深度测试等。如下表：

| 状态名称 |                          设置指令                           |                 解释                 |
| :------: | :---------------------------------------------------------: | :----------------------------------: |
|   Cull   |                    Cull Back\|Front\|Off                    | 设置剔除模式：剔除背面/正面/关闭剔除 |
|  ZTest   | ZTest Less Greater\|LEqual\|GEqual\|Equal\|NotEqual\|Always |       设置深度测试时使用的函数       |
|  ZWrite  |                       ZWrite On\|Off                        |          开启/关闭深度写入           |
|  Blend   |                  Blend SrcFactor DstFactor                  |          开启并设置混合模式          |





## SubShader中的标签

​	SubShader的标签（Tags）是一个键值对（Key/Value Pair），它的键和值都是字符串类型。这些键值对是SubShader和渲染引擎之间的沟通桥梁。它们用来告诉Unity的渲染引擎：我希望怎样以及何时渲染这个对象。

标签的结构如下：

```C
Tags { "TagName1" = "Value1" "TagName2" = "Value2" }
```

​	

SubShader的标签块支持的标签类型下如表所示：

|       标签类型       |                             说明                             |                   例子                   |
| :------------------: | :----------------------------------------------------------: | :--------------------------------------: |
|        Queue         | 控制渲染顺序，指定该物体属于哪一个渲染队列，通过这种方式可以保证所有的透明物体可以在所有不透明物体后面被渲染，我们也可以自定义使用的渲染队列来控制物体的渲染顺序 |     Tags { "Queue" = "Transparent" }     |
|      RenderType      | 对着色器进行分类，例如这是一个不透明的着色器，或是一个透明的着色器等。这可以被用于着色器替换（Shader Replacement）功能，通过其他的shader来代替此shader进行渲染，前提是被替代的shader中的RenderType的值在替代的shader中存在。 |     Tags { "RenderType" = "Opaque" }     |
|   DisableBatching    | 一些SubShader在使用Unity的批处理功能时会出现问题，例如使用了模型空间下的坐标进行顶点动画。这时可以通过该标签来直接指明是否对该SubShader使用批处理 |   Tags { "DisableBatching" = "True" }    |
| ForceNoShadowCasting |           控制使用该SubShader的物体是否会投射阴影            | Tags { "ForceNoShadowCasting" = "True" } |
|   IgnoreProjector    | 如果该标签值为“True”，那么使用该SubShader的物体将不会受Projector的影响。通常用于半透明物体 |   Tags { "IgnoreProjector" = "True" }    |
|  CanUseSpriteAtlas   |  当该SubShader是用于精灵（sprites）时，将该标签设为“False”   |  Tags { "CanUseSpriteAtlas" = "False" }  |
|     PreviewType      | 指明材质面板将如何预览该材质。默认情况下，材质将显示为一个球形，我们可以通过把该标签的值设为“Plane”“SkyBox”来改变预览类型 |     Tags { "PreviewType" = "Plane" }     |



**Queue渲染队列：**

- **Background - 背景，一般天空盒之类的使用这个标签，最早被渲染。**
- **Geometry (default)  适用于大部分不透明的物体。**
- **AlphaTest -  如果Shader要使用AlphaTest功能 使用这个队列性能更高。**
- **Transparent - 这个渲染队列在AlphaTest之后，Shader中有用到Alpha Blend的（即透明度混合的），或者深入不写入的都应该放在这个队列。**
- **Overlay 最后渲染的队列，全屏幕后期的 都应该使用这个。**



**RenderType着色器替换:**

​	**一般是不透明的物体选择“Opaque”，透明的选择“Transparent”**





## pss中的标签

​	Pass同样可以设置标签，但它的标签不同于SubShader的标签。这些标签也是用于告诉渲染引擎我们希望怎样来渲染该物体。下表给出了Pass中使用的标签类型。

|    标签类型    |                             说明                             |                     例子                     |
| :------------: | :----------------------------------------------------------: | :------------------------------------------: |
|   LightMode    |            定义该Pass在Unity的渲染流水线中的角色             |     Tags { "LightMode" = "ForwardBase" }     |
| RequireOptions | 用于指定当满足某些条件时才渲染该Pass，它的值是一个由空格分隔的字符串。目前，Unity支持的选项有：SoftVegetation。在后面的版本中，可能会增加更多的选项 | Tags { "RequireOptions" = "SoftVegetation" } |

​	

除了上面普通的Pass定义外，Unity Shader还支持一些特殊的Pass，以便进行代码复用或实现更复杂的效果。

​	**UsePass：如我们之前提到的一样，可以使用该命令来复用其他Unity Shader中的Pass；**
​	**GrabPass：该Pass负责抓取屏幕并将结果存储在一张纹理中，以用于后续的Pass处理**















