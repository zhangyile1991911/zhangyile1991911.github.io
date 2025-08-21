---
title: Unity6 深度に関する
author: zhangyile
date: 2025-5-08 09:42:00 +0800
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

# 前提
PostProcessのDepth Of Fieldを用いて手前の風景をぼかししようと思っていました。

## 原理
ランダリング最後の段階でシーン中のオブジェクトの深度により、ぼかしを行います

## 問題
![Desktop View](company/render_z_1.png){: width="875" height="482" .w-75 .normal}
1. 左のボックスと右のボックスは同じZ-Posなのに　左しかがぼかしされませんでした。（左はUniversal Render Pipeline/Unilitを利用しています）
2. DebugFrameを用いてランダリングの流れを調査します

![Desktop View](company/render_z_2.png){: width="875" height="482" .w-75 .normal}
1. DrawDepthNormalPrepass段階で物の深度をDepth Textureに書き込みます
    しかし上記のスクショットを見ると二つものしか見えないです。
    右のボックスが深度を書き込んていません
    だから　PostProcessのぼかし段階で深度値を取得出来ないので正しく表現出来なくなります。
    右ボックスのShaderを開いて見るとZWriteとZTestを追加していません
    
2. 右ボックスのShaderにZWriteとZTestを追加してみます
    前と同じ結果になりました。Unity6のURPは昔と違いところは描画時間を減少するため、Opacity Objectを描く前にDepth Textureを生成されておき、もしOpacity Objectが重ねている場合に見えない部分を描かないようにして処理流れを変更されました。
    
3. Depth Passを追加します

```
Pass
{
    Name "DepthNormalsOnly"
    Tags
    {
        "LightMode" = "DepthNormalsOnly"
    }

    // -------------------------------------
    // Render State Commands
    ZWrite On
    ZTest LEqual
    ColorMask 0
    HLSLPROGRAM
    #pragma target 2.0

    // -------------------------------------
    // Shader Stages
    #pragma vertex DepthNormalsVertex
    #pragma fragment DepthNormalsFragment

    // -------------------------------------
    // Material Keywords
    #pragma shader_feature_local _ALPHATEST_ON

    // -------------------------------------
    // Universal Pipeline keywords
    #pragma multi_compile_fragment _ _GBUFFER_NORMALS_OCT // forward-only variant
    #pragma multi_compile _ LOD_FADE_CROSSFADE
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/RenderingLayers.hlsl"

    //--------------------------------------
    // GPU Instancing
    #pragma multi_compile_instancing
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

    // -------------------------------------
    // Includes
    #include "Packages/com.unity.render-pipelines.universal/Shaders/UnlitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/UnlitDepthNormalsPass.hlsl"
    ENDHLSL
}
```

![Desktop View](company/render_z_3.png){: width="875" height="482" .w-75 .normal}

> 右ボックスは正しく深度を書き込んでぼかしされました。

![Desktop View](company/render_z_4.png){: width="875" height="482" .w-75 .normal}

## マトメ
- DirectX/Metal/Vulkan 深度值Near=1.0 Far = 0
- OpenGL/WebGL 深度值Near=0 Far=1
- Unityで統一されます
    Near=0 Far = 1
    float linearDepth = Linear01Depth(depth, _ZBufferParams); // 0（近い）→1（遠い）になります
- 不透明な物のレンダリング順番は近いから遠いまで
透明な物が遠いから近いまで