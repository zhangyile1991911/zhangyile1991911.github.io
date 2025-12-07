---
title: Unityで地面を下向きに巻き表現
author: zhangyile
date: 2025-5-18 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

Unity6 6000.0.32f1
URP 17.0.3

![Desktop View](company_without/grounddownmesh.jpg){: width="875" height="482" .w-75 .normal}
![Desktop View](company_without/grounddowncolor.jpg){: width="875" height="482" .w-75 .normal}

```
float3 ground_down(float3 positionOS,
    float3 cameraWorldPos,
    float _DownStart,
    float _DownAmplify,
    float _EnableDown)
{
    float3 positionWS = TransformObjectToWorld(positionOS);
                
    float maxDownValue = 50;
    float distanceToCamera = abs(positionWS.z - cameraWorldPos.z);
    float down = smoothstep(_DownStart, (_DownStart + maxDownValue) * _DownAmplify, distanceToCamera);
    float downValue = clamp(0, maxDownValue, distanceToCamera - _DownStart);
    positionWS.y -= downValue * down * _EnableDown;
                
    return positionWS;
}
```

> 基础知识 模型空间->世界空间->裁剪空间(相机)->视空间->NDC->屏幕空间

> 基本な知識　Model Space/Object Space -> WorldSpace -> ClipSpace -> ViewSpace -> NDC -> ScreenSpace

```
Varyings vert (Attributes input)
{
  Varyings output = (Varyings)0;
  UNITY_SETUP_INSTANCE_ID(input);
  UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);
  UNITY_SKINNED_VERTEX_COMPUTE(input);

  float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);
  positionWS = UnityFlipSprite(positionWS, unity_SpriteProps.xy);
  positionWS = ground_down(input.positionOS.xyz,_DownStart,_DownAmplify,_EnableDown)
  
  output.positionHCS = TransformWorldToHClip(positionWS);
  output.uv = TRANSFORM_TEX(input.uv, _MainTex);

  return output;
}
```