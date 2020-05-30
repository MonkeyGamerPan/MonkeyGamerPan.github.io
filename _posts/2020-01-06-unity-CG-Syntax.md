---
layout: post
title: "Unity CG Syntax"
featured-img: shane-rounce-205187
categories: [unity, CG]
---

## 关于unity中的Cg的基本语法和使用

​	Cg是类似于C语言的发展起来的图形变成语言，Cgraphrics，它的很多表达式if...else...和C语言十分相似，也和C#十分相似。

​	由于Shader是写给显卡执行的，所以没有输出语句来调试，很多地方都调试不了，只能靠Unity编辑器来帮我报错，写起来一定要小心翼翼。





## 基本类型表达式

1:语法和C语言类似,有对应的编译器,程序是给显卡运行;

2: 可以从渲染流水线中获得对应的输入;

3: 指定的输出能流入下一个流水线模块;

4: 操作符号和C语言一样，可以使用 +, -, * / <, >, <=, >= 等运算;

5: Cg提供了float half double 浮点类型;单精度，半精度，双精度

7: Cg 支持定点数 fixed来高效处理 某些小数;

8: Cg使用int来表示整数;

9: bool 数据类型来表示逻辑类型;

10:sampler*,纹理对象的句柄, sampler/1D/2D/3D/CUBE/RECT，往shader里面关联一个图片，关联纹理Texture，使用sampler

11: 内置向量数据类型: float4(float, float, float, float), 向量长度不能超过4;

12: 内置矩阵数据类型: float1x1 float2x3 float4x3 float4x4;不能超过4x4;

13: 数组类型float a[10]; 10个float, float4 b[10], 10个float4;

14: 语义绑定 float4 a : POSITION,返回值也可以语义绑定;获得上一个工位的数据和流向下一个工位的输出。只有绑定语义如Position，上一个工位才知道要传顶点的位置给它。



​	fixed定点数：如有一个32bit的Int，我要把前面16位表示整数，后面16位表示小数，用来存放35.4这个数，这个数据类型叫做定点数。实质是用整数表示小数，因为35.4输出的时候是一个比较大的整数，而它的内容里面可以表示小数。

​	定点数的加减法实际上也就是整数的加减法，性能比浮点数好，颜色就是用fixed类型表示fixed4。





## 结构体与定义

1: struct name {
	类型 名字;
	// 尽量不要使用;
	返回值 函数名称(参数) { // 如果成员函数里面使用，数据成员，该成员定义在函数前;
	}
};



2:输入语义与输出语义:
语义: 一个阶段处理数据，然后传输给下一个阶段，那么每个阶段之间的接口, 例如：顶点处理器的输入数据是处于模型空间的顶点数据（位置、法向量），输出的是投影坐标和光照颜色；片段处理器要将光照颜色做为输入;C/C++用指针，而Cg通过语义绑定的形式;
输入语义: 绑定接收参数,从上一个流水线获得参数;
输出语义: 绑定输出参数到下一个流水线模块;
语义: 入口函数上有意义(顶点着色入口,像素着色入口)，普通的函数无意义;





## 常用语义修饰

1:POSITION : 位置

2:TANGENT : 切线

3: NORMAL: 法线

4: TEXCOORD0: 第一套纹理

5: TEXCOORD1: 第二套纹理

6: TEXCOORD2: 第三套纹理

7: TEXCOORD3: 第四套纹理

8: COLOR: 颜色





## 标准内置函数

1:abs(num)绝对值;

2: 三角函数;

3: cross(a, b) 两个向量的叉积;

4: determinant(M)矩阵的行列式;

5: dot(a, b) 两个向量的点积;

6: floor(x)向下取整;

7: lerp(a, b, f), 在a, b之间线性插值;

8: log2(x) 基于2为底的x的对数;

9: mul(m, n): 矩阵x矩阵, 矩阵x向量, 向量x矩阵;

10: power(x, y) x的y次方;

11: radians(x) 度转弧度;

12: reflect(v, n) v 关于法线n的反射向量;

13: round(x) 靠近取整;

14: tex2D(smapler, x) 二维纹理查找

15: tex3Dproj(smapler, x) 投影三维纹理查找;

16: texCUBE 立方体贴图纹理查找;

 



## Unity自带函数

1: 引用Unity自带的函数库: #include “UnityCG.cginc” Unity-->Edit-->Data-->CGIncludes;

2: TRANSFORM_TEX: 根据顶点的纹理坐标，计算出对应的纹理的真正的UV坐标;

3: 使用属性的变量: 在shader里面需要使用属性变量还需要在shader中定义一下这个变量的类型和名字;名字要保持一致;

4: 外部修改shader的编辑器上的参数值;

 

例子：

1.使用编辑器的颜色对物体进行着色

```
Shader "Custom/MyShader" {
    // 属性，可以在编辑器里面bind和修改的;
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }

    SubShader {
        
        Pass {
            CGPROGRAM // 插入Cg代码开始
            fixed4 _Color; //必须定义同样的变量才能使用它

            #pragma vertex my_vert // 把my_vert作为顶shader的入口
            // 怎么样获得这个上一个工位的参数; -->语义bind
            float4 my_vert(float4 pos : POSITION) : POSITION {
                return mul(UNITY_MATRIX_MVP, pos);
            }

            #pragma fragment my_frag
            fixed4 my_frag() : COLOR{
                return _Color;//使用编辑器选择的颜色，着色物体
            }
            ENDCG // 插入Cg代码结束
        }
    }

    FallBack "Diffuse"
}
```



2.使用编辑器的纹理对物体进行绘制

```
Shader "Custom/MyShader" {
    // 属性，可以在编辑器里面bind和修改的;
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }

    SubShader {
        
        Pass {
            CGPROGRAM // 插入Cg代码开始
            //fixed4 _Color; // 定义同样的变量
            sampler2D _MainTex; // 定义同样名字的变量;


            #pragma vertex my_vert // 把my_vert作为顶shader的入口
            // 怎么样获得这个上一个工位的参数; -->语义bind
            float4 my_vert(float4 pos : POSITION) : POSITION {
                return mul(UNITY_MATRIX_MVP, pos);
            }

            #pragma fragment my_frag
            fixed4 my_frag(float2 uv : TEXCOORD0) : COLOR{　　　　　　
                //return _Color;//使用编辑器选择的颜色，着色物体
                return tex2D(_MainTex, uv);//根据uv来查到纹理并返回，我的理解是返回的东西就帮我们绘制了。uv---->_MainTex---->tex2D---->放回tex2D
            }
            ENDCG // 插入Cg代码结束
        }
    }

    FallBack "Diffuse"
}
```

编写好代码后把贴图拖进材质球MyShader的纹理属性中，视图中就显示出对应的纹理贴图。





3.定义结构体和函数来使用

```
Shader "Custom/MyShader" {
    // 属性，可以在编辑器里面bind和修改的;
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }

    SubShader {
        
        Pass {
            CGPROGRAM // 插入Cg代码开始
            struct my_struct {
                int a;
            };

            float sum(float a, float b) {
                return a + b;
            }

            fixed4 _Color; // 定义同样的变量

            #pragma vertex my_vert // 把my_vert作为顶shader的入口
            // 怎么样获得这个上一个工位的参数; -->语义bind
            float4 my_vert(float4 pos : POSITION) : POSITION {
                return mul(UNITY_MATRIX_MVP, pos);
            }

            #pragma fragment my_frag
            fixed4 my_frag() : COLOR{
                return _Color;
            }
            ENDCG // 插入Cg代码结束
        }
    }
    FallBack "Diffuse"
}
```





4.语义绑定可以绑定一个结构体，在结构体里面再指定哪个属性。Unity其实给我们定义了很多常用的结构体，可以直接使用，在 “UnityCG.cginc” Unity-->Edit-->Data-->CGIncludes;里面查看

```
Shader "Custom/MyShader" {
    // 属性，可以在编辑器里面bind和修改的;
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }

    SubShader {
        
        Pass {
            CGPROGRAM // 插入Cg代码开始

            fixed4 _Color; // 定义同样的变量

　　　　　　　//结构体包装语义
            struct my_vert_data {
                float4 pos : POSITION;
            };

            #pragma vertex my_vert // 把my_vert作为顶shader的入口
            // 怎么样获得这个上一个工位的参数; -->语义bind
            float4 my_vert(my_vert_data data) : POSITION {
                return mul(UNITY_MATRIX_MVP, data.pos);
            }

            #pragma fragment my_frag
            fixed4 my_frag(float2 uv : TEXCOORD0) : COLOR{
                return _Color;
            }
            ENDCG // 插入Cg代码结束
        }
    }
    FallBack "Diffuse"
}
```



## 使用C#脚本来控制和编写shader

0.shader文件里面的代码是这样的：

```
Shader "Custom/MyShader" {
    // 属性，可以在编辑器里面bind和修改的;
    Properties {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }

    SubShader {
        
        Pass {
            CGPROGRAM // 插入Cg代码开始
            fixed4 _Color; //必须定义同样的变量才能使用它

            #pragma vertex my_vert // 把my_vert作为顶shader的入口
            // 怎么样获得这个上一个工位的参数; -->语义bind
            float4 my_vert(my_vert_data data) : POSITION {
                return mul(UNITY_MATRIX_MVP, data.pos);
            }

            #pragma fragment my_frag
            fixed4 my_frag() : COLOR{
                return _Color;//使用编辑器选择的颜色，着色物体
            }
            ENDCG // 插入Cg代码结束
        }
    }

    FallBack "Diffuse"
}
```



1.创建一个叫shader_ctrl的脚本

2.挂载到Cube节点下，这个Cube的材质属性就是Myshader材质球，Myshader材质球的shader属性是MyShader.shader文件

3.打开shader_ctrl的脚本，内容如下(记得关联public 材质属性)：



```c#
using UnityEngine;
using System.Collections;

public class shader_ctrl : MonoBehaviour {
    public Material material;
    // Use this for initialization
    void Start () {
        material.SetColor("_Color", new Color(1.0f, 0.0f, 0.0f, 1.0f));//修改颜色为红色
    }
    
    // Update is called once per frame
    void Update () {
    
    }
}
```

4.运行，发现cube变红色





















