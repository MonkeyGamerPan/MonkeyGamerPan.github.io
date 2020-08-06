---
layout: post
title: "Unity Web Address"
featured-img: shane-rounce-205187
categories: [unity, address]
---
<br/>
<br/>
[unity常见面试题](https://blog.csdn.net/qq_25601345/article/details/77102775)
<br/>
<br/>
Gameplay Ingredients框架 
<br/>
<br/>


[Unity——Bit编程，可用于计算技能解锁\装备购买并初步加密，降低内存占有量并提高安全性](https://blog.csdn.net/Htlas/article/details/79717990)
<br>
<br>
[Unity Shader学习笔记 - 用UV动画实现沙滩上的泡沫](https://blog.csdn.net/ltycloud/article/details/79417637)

<br/>

<br/>

unity中的前段以及后端框架，ET框架和GF框架，都是开源，各有各的优缺点
<br/>
<br/>
unity中的热跟新，现在有使用Lua和ILRuntime框架，

<br/>
<br/>
unity中的UI框架可以学习PureMVC框架

<br/>

<br/>

unity中随机生成地图，像饥荒中的那样，使用   波函数坍缩算法，WFC算法   


ml-agents unity中的机器学习。

DOTS是什么？
DOTS是给几项优化技术打包起了个名字，主要还是为了宣传考虑。它主要包含3个技术：
	ECS模式。这种编程模式，要求游戏逻辑和引擎功能，都要按照 实体+组件+系统 的框架构建。采用ECS模式后，可以得到性能与多线程并行的优化。
	JobSystem。一套先进的多线程任务管理系统，搭配ECS时可以获得最佳性能。不搭配ECS也能在引擎的基本功能方面享受到好处。
	Burst，可以代替.Net原来的运行时环境（Mono或IL2CPP），用全新的编译器编译为原生代码，得到比IL2CPP更快的执行速度。当然代码最好能配合Burst做一些代码上的改动，以得到充分优化、避免BUG  ：）
  DOTS系统成熟以后，游戏的脚本逻辑部分会和现有的组件式系统完全不同。原有脚本写法和ECS写法的区别，比两种语言的差别要大得多。所以起个好名字逐步发展新体系也是必要的。




ECS是什么？
	首先ECS并非一种全新的技术，也不是Unity首先提出的。这种技术的出现非常早，而近几年突然火爆，是因为暴雪的《守望先锋》。《守望先锋》的服务器和客户端框架完全基于ECS构建，在游戏机制、网络、渲染方面都有非常出色的表现，这种技术上的巨大成功，必然会引领一波ECS的潮流。

	ECS不属于某种“设计模式”，之前我们常说的“设计模式”是在面向对象的框架之下讨论的，而ECS完全是另一种面向对象的编程方法，它对传统编程方法的颠覆性比Actor模式更强。至少目前来看，ECS仅适合于游戏开发领域。
	
	E代表实体（Entity）。实体在逻辑意义上是组件的容器，每个实体有若干个组件；但在实现上它只需要一个整数ID，区分不同的实体。
	C代表组件（Component），它不是目前Unity中的组件，而是一种只拥有数据，不能具有任何方法的结构体。
	S代表系统（System），和C相反，系统只能具有方法而不具有任何数据。

参考：http://www.u3dc.com/archives/3509


[【Unity】TimeLine&Cinemachine系列教程——动态赋值，我要打十个！](https://www.bilibili.com/read/cv29363/)



[unity高清渲染管线入门指南](https://connect.unity.com/p/gao-qing-xi-xuan-ran-guan-xian-hdrpru-men-zhi-nan)



[unity使用shader graph 制作溶解特效](https://connect.unity.com/p/li-yong-shadergraphzhao-se-qi-shi-tu-shi-xian-xuan-ku-de-wu-ti-xiao-rong-te-xiao)



[unity shader graph Position节点解析](https://www.bilibili.com/read/cv3585934/)



[unity UI框架](http://www.manew.com/thread-42748-1-1.html)



[unity 用户手册](https://docs.unity3d.com/Manual/UnityManual.html)



[unity 社区connect](https://connect.unity.com)



[unity AssetBundle讲解](https://blog.csdn.net/qq_35361471/article/details/82854560#1_AssetBundle_3)





游戏小组空间 ：http://renxufeng.ys168.com/

unity shader 入门精要 冯乐乐 github 博客地址: http://candycat1992.github.io/portfolio/
unity shader 入门精要 配套资源github地址：https://github.com/candycat1992/Unity_Shaders_Book

##### Unity Gerstner Waves水面波浪:用于模拟水的波浪



[Unity3D教程：镜面反射原理及实现（一）](https://gameinstitute.qq.com/community/detail/106151)



[unity3d 实时平面反射的拓展和应用](https://zhuanlan.zhihu.com/p/37648960)



[unity URP移动平台的屏幕空间反射](https://zhuanlan.zhihu.com/p/150890059)



[Unity3D水特效之雨天模拟（一）](https://zhuanlan.zhihu.com/p/37796757)



[MaxwellGeng 一只二哈（知乎大佬首页）](https://www.zhihu.com/people/maxwellgeng)





[Unity3D-Shader-热扭曲效果](https://www.cnblogs.com/lijiajia/p/6861516.html)



使用脚本将dds贴图转换成png格式的图片：

```c#
[MenuItem("Assets/一键替换DDS的贴图")]
    public static void ToChangeMaterialsDDS()
    {
        //获取选中目录下的所有Material类型文件对象
        UnityEngine.Object[] m_objects = Selection.GetFiltered(typeof(Material), SelectionMode.DeepAssets);//选择的所有对象
        //遍历所有材质
        foreach (UnityEngine.Object item in m_objects)
        {
          
            if (Path.GetExtension(AssetDatabase.GetAssetPath(item)) != "")//判断路径是否为空
            {
                string path = AssetDatabase.GetAssetPath(item);
                string oldTextruePath = AssetDatabase.GetAssetPath(((Material)item).mainTexture);
                //判断材质的mainTexture是否为.dds格式
                if (AssetDatabase.GetAssetPath(((Material)item).mainTexture).Contains(".dds"))
                {
                    //如果为.dds格式，获取其同名.png文件路径
                    string newTextruePath = AssetDatabase.GetAssetPath(((Material)item).mainTexture).Replace(".dds", ".png");
                    if (Path.GetExtension(newTextruePath) != "")//判断同目录下是否有同名.png文件
                    {
                        //则将材质的mainTexture改为转换好的同目录下的.png格式贴图，编辑器模式下使用AssetDatabase.LoadAssetAtPath读取资源
                        ((Material)item).mainTexture = AssetDatabase.LoadAssetAtPath<Texture>(newTextruePath);
                        //替换成功后删除.dds格式的贴图文件
                        AssetDatabase.DeleteAsset(oldTextruePath);
                        Debug.Log(AssetDatabase.GetAssetPath(item) + "TextureName=" + AssetDatabase.GetAssetPath(((Material)item).mainTexture));
                    }
                }
              
            }

        }
        //保存并刷新资源
        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();
    }

```













