---
title: Unity6でデカール(Decal)
author: zhangyile
date: 2025-08-30 13:42:00 +0800
categories: [Work Log]
tags: [Work,Development,Decal,Custom Shader]
comments: false
img_path: /assets/img/
image:
  path: company_without/decal_test7.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---


ChatGPTにより

描画順

| デカール方式 | 描画タイミング | 備考 |
| --- | --- | --- |
| **DBuffer Decal** | 不透明物体の描画**前** | メッシュに焼き込む／GBuffer的 |
| **Screen Space Decal** | 不透明描画**後**, 透明描画**前** | スクリーン上にオーバーレイ |


## 試し

> Unityで基礎なオブジェクトを使って試しました

![Desktop View](company_without/decal_test1.jpg){: width="642" height="425" .w-75 .normal}


## Unlit Custom Shaderで試し

> 一番簡単なShade Codeを生成しました

```
Shader "Custom/MyCustomShader"
{
    Properties
    {
        [MainColor] _BaseColor("Base Color", Color) = (1, 1, 1, 1)
        [MainTexture] _BaseMap("Base Map", 2D) = "white"
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" }

        Pass
        {
            HLSLPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct Attributes
            {
                float4 positionOS : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct Varyings
            {
                float4 positionHCS : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

            TEXTURE2D(_BaseMap);
            SAMPLER(sampler_BaseMap);

            CBUFFER_START(UnityPerMaterial)
                half4 _BaseColor;
                float4 _BaseMap_ST;
            CBUFFER_END

            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
                OUT.uv = TRANSFORM_TEX(IN.uv, _BaseMap);
                return OUT;
            }

            half4 frag(Varyings IN) : SV_Target
            {
                half4 color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv) * _BaseColor;
                return color;
            }
            ENDHLSL
        }
    }
}

```

- 青い立方体にビルドインシェーダーがアタッチしています
- 赤い立方体に上記のCustom Shaderがアタッチしています

![Desktop View](company_without/decal_test2.jpg){: width="642" height="425" .w-75 .normal}

> 結果から見ると赤い立方体にテクスチャがくっついていません。なぜ？


## Custom Shaderにデカールを有効にする

1. 分析 Decalはいつに発生します
  - フレームデバッグを使って描画順番から分析します

![Desktop View](company_without/decal_test3.jpg){: width="642" height="425" .w-75 .normal}

> 上記のスクリーンショットにより、不透明なオブジェクトが描画される前に発生します

2. コードで何を漏らしました
  - フレームデバッグにより,UnityのLit.shaderがもう一つことをやっています

![Desktop View](company_without/decal_test4.jpg){: width="642" height="425" .w-75 .normal}

> この段階でシーン内のオブジェクトの法線を出力しました

3. Litコードを読みます
  - 法線情報をGBufferに書き込みます
  - フレームデバッグにより、上向き色は緑で、右向きは赤いです。
  - 色データはRGBAで構成されたので、XYZ -> RGB

```
Pass
{
  Name "DepthNormals"

  Tags
  {
    "LightMode" = "DepthNormals"
  }

  // -------------------------------------
  // Render State Commands
  ZWrite On
  Cull[_Cull]

  HLSLPROGRAM
  #pragma target 2.0

  // -------------------------------------
  // Shader Stages
  #pragma vertex DepthNormalsVertex
  #pragma fragment DepthNormalsFragment

  // -------------------------------------
  // Material Keywords
  #pragma shader_feature_local _NORMALMAP
  #pragma shader_feature_local _PARALLAXMAP
  #pragma shader_feature_local _ _DETAIL_MULX2 _DETAIL_SCALED
  #pragma shader_feature_local _ALPHATEST_ON
  #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

  // -------------------------------------
  // Unity defined keywords
  #pragma multi_compile _ LOD_FADE_CROSSFADE

  // -------------------------------------
  // Universal Pipeline keywords
  #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/RenderingLayers.hlsl"

  //--------------------------------------
  // GPU Instancing
  #pragma multi_compile_instancing
  #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

  // -------------------------------------
  // Includes
  #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
  #include "Packages/com.unity.render-pipelines.universal/Shaders/LitDepthNormalsPass.hlsl"
  ENDHLSL
}
```

4. 法線を書き込むコードをコーピしてCustom Shaderに貼り付けます
  - 法線情報が書き込まれるようになりました

![Desktop View](company_without/decal_test5.jpg){: width="642" height="425" .w-75 .normal}

5. なぜ　テクスチャーがくっ付いていません
  - まだ　何のコードを漏らしましたか
  - Litのコードを読み続けます
  - ForwardLitのLitPassFragmentを掘り下げます
  - LitFowardPass.hlslにApplyDecalToSurfaceDataを書いてあります
  - DBuffer.hlsl内で四つDecal関数があります

![Desktop View](company_without/decal_test6.jpg){: width="642" height="425" .w-75 .normal}

> Lit.shaderのPass"ForwardLit"

```
Pass
{
  Name "ForwardLit"
  Tags
  {
    "LightMode" = "UniversalForward"
  }
  #pragma vertex LitPassVertex
  #pragma fragment LitPassFragment
}
```

> LitFowardPass.hlsl

```
void LitPassFragment(
    Varyings input
    , out half4 outColor : SV_Target0
)
{
#if defined(_DBUFFER)
    ApplyDecalToSurfaceData(input.positionCS, surfaceData, inputData);
#endif
}
```

> DBuffer.hlslで四つApplyDecal関数が提供しています

```
void ApplyDecalToBaseColor(float4 positionCS, inout half3 baseColor)

void ApplyDecal(float4 positionCS,
    inout half3 baseColor,
    inout half3 specularColor,
    inout half3 normalWS,
    inout half metallic,
    inout half occlusion,
    inout half smoothness)

void ApplyDecalToBaseColorAndNormal(float4 positionCS, inout half3 baseColor, inout half3 normalWS);

void ApplyDecalToBaseColor(float4 positionCS, inout half3 baseColor);

```

6. 手動でApplyDecalを呼びます
  - DBuffer.hlslを導入します
  - 法線を換算(Unlitなのでしなくても大丈夫です)
  - fragment段階でApplyDecal関数を呼びます
```
Pass
{
    HLSLPROGRAM

    #pragma vertex vert
    #pragma fragment frag

    #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DBuffer.hlsl"//ファイルを導入
    struct Attributes
    {
        float4 positionOS : POSITION;
        float3 normalOS : NORMAL;
        float2 uv : TEXCOORD0;
    };

    struct Varyings
    {
        float4 positionHCS : SV_POSITION;
        float3 normalWS : NORMAL;
        float2 uv : TEXCOORD0;
    };

    TEXTURE2D(_BaseMap);
    SAMPLER(sampler_BaseMap);

    CBUFFER_START(UnityPerMaterial)
        half4 _BaseColor;
        float4 _BaseMap_ST;
    CBUFFER_END

    Varyings vert(Attributes IN)
    {
        Varyings OUT;
        OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
        OUT.uv = TRANSFORM_TEX(IN.uv, _BaseMap);
        //法線の換算
        OUT.normalWS = TransformObjectToWorldNormal(IN.normalOS.xyz);
        return OUT;
    }

    half4 frag(Varyings IN) : SV_Target
    {
        half4 color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv) * _BaseColor;
        //デカールの色をかけます
        ApplyDecalToBaseColorAndNormal(IN.positionHCS,color.xyz,IN.normalWS);
        return color;
    }
    ENDHLSL
}
```

![Desktop View](company_without/decal_test7.jpg){: width="642" height="425" .w-75 .normal}

## 纏り
  1. オブジェクトが深度と表面法線をGBufferに書き込む必要があります
  2. Custom Shaderに手動でApplyDecal関数を呼ぶ必要があります





