---
layout: post
title: "Unity shader 内置变量"
featured-img: shane-rounce-205187
categories: [unity, shader]
---

​	使用Unity写Shader的一个好处在于，它提供了很多内置的参数，这使得我们不再需要自己手动计算一些值。本节将给出Unity内置的用于空间变换和摄像机以及屏幕参数的内置变量。这些内置变量可以在UnityShaderVariables.cginc文件中找到定义和说明。



## 变换矩阵

​	首先是用于坐标空间变换的矩阵。下表给出了Unity 5.2版本提供的所有内置变换矩阵。下面所有的矩阵都是float4x4类型的。

|       变量名       |                             描述                             |
| :----------------: | :----------------------------------------------------------: |
|  UNITY_MATRIX_MVP  | 当前的模型观察投影矩阵，用于将顶点/方向矢量从模型空间变换到裁剪空间 |
|  UNITY_MATRIX_MV   | 当前的模型观察矩阵，用于将顶点/方向矢量从模型空间变换到观察空间 |
|   UNITY_MATRIX_V   | 当前的观察矩阵，用于将顶点/方向矢量从世界空间变换到观察空间  |
|   UNITY_MATRIX_P   | 当前的投影矩阵，用于将顶点/方向矢量从观察空间变换到裁剪空间  |
|  UNITY_MATRIX_VP   | 当前的观察投影矩阵，用于将顶点/方向矢量从世界空间变换到裁剪空间 |
| UNITY_MATRIX_T_MV  |                  UNITY_MATRIX_MV的转置矩阵                   |
| UNITY_MATRIX_IT_MV | UNITY_MATRIX_MV的逆转置矩阵，用于将法线从模型空间变换到观察空间，也可用于得到UNITY_MATRIX_MV的逆矩阵 |
|   _Object2World    | 当前的模型矩阵，用于将顶点/方向矢量从模型空间变换到世界空间  |
|   _World2Object    | _Object2World的逆矩阵，用于将顶点/方向矢量从世界空间变换到模型空间 |



​	其中有一个矩阵比较特殊，即UNITY_MATRIX_T_MV矩阵。很多对数学不了解的读者不理解这个矩阵有什么用处。如果读者认真看过矩阵一节的知识，应该还会记得一种非常吸引人的矩阵类型——正交矩阵。对于正交矩阵来说，它的逆矩阵就是转置矩阵。因此，如果UNITY_MATRIX_MV是一个正交矩阵的话，那么UNITY_MATRIX_T_MV就是它的逆矩阵，也就是说，我们可以使用UNITY_MATRIX_T_MV把顶点和方向矢量从观察空间变换到模型空间。那么问题是，UNITY_MATRIX_MV什么时候是一个正交矩阵呢？读者可以从4.5节找到答案。总结一下，如果我们只考虑旋转、平移和缩放这3种变换的话，如果一个模型的变换只包括旋转，那么UNITY_MATRIX_MV就是一个正交矩阵。这个条件似乎有些苛刻，我们可以把条件再放宽一些，如果只包括旋转和统一缩放（假设缩放系数是k），那么UNITY_MATRIX_MV就几乎是一个正交矩阵了。为什么是几乎呢？因为统一缩放可能会导致每一行（或每一列）的矢量长度不为1，而是k，这不符合正交矩阵的特性，但我们可以通过除以这个统一缩放系数，来把它变成正交矩阵。在这种情况下，UNITY_MATRIX_MV的逆矩阵就是\frac{1}{k}UNITY_MATRIX_T_MV。而且，如果我们只是对方向矢量进行变换的话，条件可以放得更宽，即不用考虑有没有平移变换，因为平移对方向矢量没有影响。因此，我们可以截取UNITY_MATRIX_T_MV的前3行前3列来把方向矢量从观察空间变换到模型空间（前提是只存在旋转变换和统一缩放）。对于方向矢量，我们可以在使用前对它们进行归一化处理，来消除统一缩放的影响。

​	还有一个矩阵需要说明一下，那就是UNITY_MATRIX_IT_MV矩阵。我们在4.7节已经知道，法线的变换需要使用原变换矩阵的逆转置矩阵。因此UNITY_MATRIX_IT_MV可以把法线从模型空间变换到观察空间。但只要我们做一点手脚，它也可以用于直接得到UNITY_MATRIX_MV的逆矩阵——我们只需要对它进行转置就可以了。因此，为了把顶点或方向矢量从观察空间变换到模型空间，我们可以使用类似下面的代码：

```c
// 方法一：使用transpose函数对UNITY_MATRIX_IT_MV进行转置，
// 得到UNITY_MATRIX_MV的逆矩阵，然后进行列矩阵乘法，
// 把观察空间中的点或方向矢量变换到模型空间中
float4 modelPos = mul(transpose(UNITY_MATRIX_IT_MV), viewPos);

// 方法二：不直接使用转置函数transpose，而是交换mul参数的位置，使用行矩阵乘法
// 本质和方法一是完全一样的
float4 modelPos = mul(viewPos, UNITY_MATRIX_IT_MV);
```





## 摄像机和屏幕参数

|             变量名             |   类型   |                             描述                             |
| :----------------------------: | :------: | :----------------------------------------------------------: |
|      _WorldSpaceCameraPos      |  float3  |                  该摄像机在世界空间中的位置                  |
|       _ProjectionParams        |  float4  | x = 1.0（或−1.0，如果正在使用一个翻转的投影矩阵进行渲染），y = Near，z = Far，w = 1.0 + 1.0/Far，其中Near和Far分别是近裁剪平面和远裁剪平面和摄像机的距离 |
|         _ScreenParams          |  float4  | x = width，y = height，z = 1.0 + 1.0/width，w = 1.0 + 1.0/height，其中width和height分别是该摄像机的渲染目标（render target）的像素宽度和高度 |
|         _ZBufferParams         |  float4  | x = 1− Far/Near，y = Far/Near，z = x/Far，w = y/Far，该变量用于线性化Z缓存中的深度值 |
|       unity_OrthoParams        |  float4  | x = width，y = heigth，z没有定义，w = 1.0（该摄像机是正交摄像机）或w = 0.0（该摄像机是透视摄像机），其中width和height是正交投影摄像机的宽度和高度 |
|     unity_CameraProjection     | float4x4 |                      该摄像机的投影矩阵                      |
|    unity_CameraInvProjectio    | float4x4 |                  该摄像机的投影矩阵的逆矩阵                  |
| unity_CameraWorldClipPlanes[6] | float4x4 | 该摄像机的6个裁剪平面在世界空间下的等式，按如下顺序：左、右、下、上、近、远裁剪平面 |





















































































































