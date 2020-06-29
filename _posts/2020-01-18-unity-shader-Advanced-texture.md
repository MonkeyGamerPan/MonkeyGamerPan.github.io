---
layout: post
title: "Unity 高级纹理"
featured-img: shane-rounce-205187
categories: [unity, shader]
---

# 									高级纹理



## 立方体纹理

​	在图形学中，**立方体纹理（Cubemap）是环境映射（Environment Mapping）的一种实现方法**。环境映射可以模拟物体周围的环境，而使用了环境映射的物体可以看起来像镀了层金属一样反射出周围的环境。

​	和之前见到的纹理不同，立方体纹理一共包含了6张图像，这些图像对应了一个立方体的6个面，立方体纹理的名称也由此而来。立方体的每个面表示沿着世界空间下的轴向（上、下、左、右、前、后）观察所得的图像。那么，我们如何对这样一种纹理进行采样呢？和之前使用二维纹理坐标不同，对立方体纹理采样我们需要提供一个三维的纹理坐标，这个三维纹理坐标表示了我们在世界空间下的一个3D方向。这个方向矢量从立方体的中心出发，当它向外部延伸时就会和立方体的6个纹理之一发生相交，而采样得到的结果就是由该交点计算而来的。下图给出了使用方向矢量对立方体纹理采样的过程。

![](../assets/img/resources/SamplerCubeMapTexture.png)

​	使用立方体纹理的好处在于，它的实现简单快速，而且得到的效果也比较好。但它也有一些缺点，例如当场景中引入了新的物体、光源，或者物体发生移动时，我们就需要重新生成立方体纹理。除此之外，立方体纹理也仅可以反射环境，但不能反射使用了该立方体纹理的物体本身。这是因为，立方体纹理不能模拟多次反射的结果，例如两个金属球互相反射的情况（事实上，Unity 5引入的全局光照系统允许实现这样的自反射效果）。由于这样的原因，想要得到令人信服的渲染结果，我们应该尽量对凸面体而不要对凹面体使用立方体纹理（因为凹面体会反射自身）。

​	**立方体纹理在实时渲染中有很多应用，最常见的是用于天空盒子（Skybox）以及环境映射**。



### 天空盒子

​	天空盒子（Skybox）是游戏中用于模拟背景的一种方法。天空盒子这个名字包含了两个信息：它是用来模拟天空的（尽管现在我们仍可以用它模拟室内等背景），它是一个盒子。当我们在场景中使用了天空盒子时，整个场景就被包围在一个立方体内。这个立方体的每个面使用的技术就是立方体纹理映射技术。

​	在Unity中，想要使用天空盒子非常简单。我们只需要创建一个Skybox材质，再把它赋给该场景的相关设置即可。为了让天空盒子正常渲染，我们需要把这6张纹理的**Wrap Mode设置为Clamp**，以防止在接缝处出现不匹配的现象

​	Skybox纹理除了6张纹理属性外还有3个属性：**Tint Color，用于控制该材质的整体颜色；Exposure，用于调整天空盒子的亮度；Rotation，用于调整天空盒子沿+y轴方向的旋转角度。**

​	为了让摄像机正常显示天空盒子，我们还需要保证渲染场景的摄像机的Camera组件中的Clear Flags被设置为Skybox。

​	**需要说明的是，在Window → Lighting → Skybox中设置的天空盒子会应用于该场景中的所有摄像机。如果我们希望某些摄像机可以使用不同的天空盒子，可以通过向该摄像机添加Skybox组件来覆盖掉之前的设置。也就是说，我们可以在摄像机上单击Component → Rendering → Skybox来完成对场景默认天空盒子的覆盖。**

​	**在Unity中，天空盒子是在所有不透明物体之后渲染的，而其背后使用的网格是一个立方体或一个细分后的球体。**





### 创建用于环境映射的立方体纹理

​	创建用于环境映射的立方体纹理的方法有三种：

​	**第一种方法是直接由一些特殊布局的纹理创建；第二种方法是手动创建一个Cubemap资源，再把6张图赋给它；第三种方法是由脚本生成。**

​	如果使用第一种方法，我们需要提供一张具有特殊布局的纹理，例如类似立方体展开图的交叉布局、全景布局等。然后，我们只需要把该纹理的Texture Type设置为Cubemap即可，Unity会为我们做好剩下的事情。在基于物理的渲染中，我们通常会使用一张HDR图像来生成高质量的Cubemap。

​	第二种方法是Unity 5之前的版本中使用的方法。我们首先需要在项目资源中创建一个Cubemap，然后把6张纹理拖曳到它的面板中。在Unity 5中，官方推荐使用第一种方法创建立方体纹理，这是因为第一种方法可以对纹理数据进行压缩，而且可以支持边缘修正、光滑反射（glossy reflection）和HDR等功能。

前面两种方法都需要我们提前准备好立方体纹理的图像，它们得到的立方体纹理往往是被场景中的物体所共用的。但在理想情况下，我们希望根据物体在场景中位置的不同，生成它们各自不同的立方体纹理。这时，我们就可以在Unity中使用脚本来创建。这是通过利用Unity提供的Camera.RenderToCubemap函数来实现的。Camera.RenderToCubemap函数可以把从任意位置观察到的场景图像存储到6张图像中，从而创建出该位置上对应的立方体纹理。





### 反射

​	使用了反射效果的物体通常看起来就像镀了层金属。想要模拟反射效果很简单，**我们只需要通过入射光线的方向和表面法线方向来计算反射方向，再利用反射方向对立方体纹理采样即可。**

示例代码如下：

```c#
Shader "Unity Shaders Book/Chapter 10/Reflection" {
	Properties {
		_Color ("Color Tint", Color) = (1, 1, 1, 1)
		_ReflectColor ("Reflection Color", Color) = (1, 1, 1, 1)
		_ReflectAmount ("Reflect Amount", Range(0, 1)) = 1
		_Cubemap ("Reflection Cubemap", Cube) = "_Skybox" {}
	}
	SubShader {
		Tags { "RenderType"="Opaque" "Queue"="Geometry"}
		
		Pass { 
			Tags { "LightMode"="ForwardBase" }
			
			CGPROGRAM
			
			#pragma multi_compile_fwdbase
			
			#pragma vertex vert
			#pragma fragment frag
			
			#include "Lighting.cginc"
			#include "AutoLight.cginc"
			
			fixed4 _Color;
			fixed4 _ReflectColor;
			fixed _ReflectAmount;
			samplerCUBE _Cubemap;
			
			struct a2v {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};
			
			struct v2f {
				float4 pos : SV_POSITION;
				float3 worldPos : TEXCOORD0;
				fixed3 worldNormal : TEXCOORD1;
				fixed3 worldViewDir : TEXCOORD2;
				fixed3 worldRefl : TEXCOORD3;
				SHADOW_COORDS(4)
			};
			
			v2f vert(a2v v) {
				v2f o;
				
				o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
				
				o.worldNormal = UnityObjectToWorldNormal(v.normal);
				
				o.worldPos = mul(_Object2World, v.vertex).xyz;
				
				o.worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
				
				// Compute the reflect dir in world space
				o.worldRefl = reflect(-o.worldViewDir, o.worldNormal);
				
				TRANSFER_SHADOW(o);
				
				return o;
			}
			
			fixed4 frag(v2f i) : SV_Target {
				fixed3 worldNormal = normalize(i.worldNormal);
				fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));		
				fixed3 worldViewDir = normalize(i.worldViewDir);		
				
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				
				fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(worldNormal, worldLightDir));
				
				// Use the reflect dir in world space to access the cubemap
				fixed3 reflection = texCUBE(_Cubemap, i.worldRefl).rgb * _ReflectColor.rgb;
				
				UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
				
				// Mix the diffuse color with the reflected color
				fixed3 color = ambient + lerp(diffuse, reflection, _ReflectAmount) * atten;
				
				return fixed4(color, 1.0);
			}
			
			ENDCG
		}
	}
	FallBack "Reflective/VertexLit"
}
```

​	对立方体纹理的采样需要使用CG的texCUBE函数。注意到，在上面的计算中，我们在采样时并没有对i.worldRefl进行归一化操作。这是因为，用于采样的参数仅仅是作为方向变量传递给texCUBE函数的，因此我们没有必要进行一次归一化的操作。然后，我们使用_ReflectAmount来混合漫反射颜色和反射颜色，并和环境光照相加后返回。

​	在上面的计算中，我们选择在顶点着色器中计算反射方向。当然，我们也可以选择在片元着色器中计算，这样得到的效果更加细腻。但是，对于绝大多数人来说这种差别往往是可以忽略不计的，因此出于性能方面的考虑，我们选择在顶点着色器中计算反射方向。





### 折射

​	折射的物理原理比反射复杂一些。我们在初中物理就已经接触过折射的定义：当光线从一种介质（例如空气）斜射入另一种介质（例如玻璃）时，传播方向一般会发生改变。当给定入射角时，我们可以使用**斯涅尔定律（Snell's Law）**来计算反射角。当光从介质1沿着和表面法线夹角为θ1的方向斜射入介质2时，我们可以使用如下公式计算折射光线与法线的夹角θ2：

​																	**η1sinθ1=η2sinθ2**

<img src="../assets/img/resources/Snell'sLaw.png" style="zoom: 25%;" />

​	其中，**η1和η2分别是两个介质的折射率（index of refraction）**。折射率是一项重要的物理常数，例如真空的折射率是1，**而玻璃的折射率一般是1.5**。

​	通常来说，当得到折射方向后我们就会直接使用它来对立方体纹理进行采样，但这是不符合物理规律的。对一个透明物体来说，一种更准确的模拟方法需要计算两次折射—— 一次是当光线进入它的内部时，而另一次则是从它内部射出时。但是，想要在实时渲染中模拟出第二次折射方向是比较复杂的，而且仅仅模拟一次得到的效果从视觉上看起来“也挺像那么回事的”。正如我们之前提到的——图形学第一准则“如果它看起来是对的，那么它就是对的”。因此，在实时渲染中我们通常仅模拟第一次折射。





























































































































