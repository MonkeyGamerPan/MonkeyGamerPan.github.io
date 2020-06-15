---
layout: post
title: "Unity 使用相机渲染到CubeMap"
featured-img: shane-rounce-205187
categories: [unity]
---

​	从这个相机渲染成一个静态立方体图。这个功能在编辑器中对于“烘焙”场景的静态立方体非常有用。参见下面的向导示例。如果您想要一个实时更新的cubemap，请使用RenderToCubemap变体，该变体使用cubemap维度的RenderTexture，见下面。摄像机的位置，清晰的旗帜和裁剪平面的距离将被用于渲染到cubemap面。faceMask是一个bitfield，指示哪个cubemap面应该被渲染成。每一个被设置的位都对应于一张脸。位数是CubemapFace枚举的整数值。默认情况下，所有六个cubemap面都将被渲染(默认值63有六个最低位)。如果向cubemap渲染失败，该函数将返回false。一些图形硬件不支持该功能。还要注意，ReflectionProbes是执行实时反射的更高级的方法。通过选择Create->遗留选项，可以在编辑器中创建Cubemaps。

```c#
using UnityEngine;
using UnityEditor;
using System.Collections;

public class RenderCubemapWizard : ScriptableWizard
{
    public Transform renderFromPosition;
    public Cubemap cubemap;

    void OnWizardUpdate()
    {
        string helpString = "Select transform to render from and cubemap to render into";
        bool isValid = (renderFromPosition != null) && (cubemap != null);
    }

    void OnWizardCreate()
    {
        // create temporary camera for rendering
        GameObject go = new GameObject("CubemapCamera");
        go.AddComponent<Camera>();
        // place it on the object
        go.transform.position = renderFromPosition.position;
        go.transform.rotation = Quaternion.identity;
        // render into cubemap
        go.GetComponent<Camera>().RenderToCubemap(cubemap);

        // destroy temporary camera
        DestroyImmediate(go);
    }

    [MenuItem("GameObject/Render into Cubemap")]
    static void RenderCubemap()
    {
        ScriptableWizard.DisplayWizard<RenderCubemapWizard>(
            "Render cubemap", "Render!");
    }
}
```

