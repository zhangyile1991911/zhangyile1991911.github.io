---
title: Unity6 霧めいた表現
author: zhangyile
date: 2025-08-20 09:42:00 +0800
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

## 前提

> 現在ぼかし方法について描画処理がとても重いので、フレームごとに各オブジェクトがぼかしを行っています。
> 例えば、一つオブジェクトが10*10ループのぼかしを行いますから、シーンに100オブジェクトが存在する場合、100passを行い、即ち100 * 10 * 10 = 10000ループします。
> pass数が多ければ多い程、処理が重くなり、フレーム数が落ち、ゲームが滑らかにならないです。

## 解決方法

> 各オブジェクトがぼかしするより、描画の最終段階(postprocess)で一括でぼかしする方により描画負荷が軽く、どれぐらいオブジェクトがあっても5,6pass処理でぼかしでき、描画処理が遊びに差し支えないようになります。

## PostProcess

1. オブジェクトの深度により、ぼかしをします
2. オブジェクトが自分の深度を_CameraDepthTextureに書き込みます
![Desktop View](company_without/dof1.png){: width="642" height="425" .w-75 .normal}

3. 上記のRender Graph Viewerの配置を見ると _CameraDepthTextureの書き込みタイミングは「Draw Depth Normal Prepass」です。
4. 現在のshaderに「Draw Depth Normal Prepass」を追加します

5. 上記のRender Graph Viewerの配置を見ると _CameraDepthTextureの書き込みタイミングは「Draw Depth Normal Prepass」です。
6. 現在のshaderに「Draw Depth Normal Prepass」を追加します

```hlsl
Pass
{
    Name "GroundDepthNormals"
    Tags
    {
        "LightMode" = "DepthNormals" //重要な設定
    }
    ZWrite On　//重要な設定
    ColorMask 0
    HLSLPROGRAM
    #pragma target 2.0

    #pragma vertex DepthNormalsVertex
    #pragma fragment DepthNormalsFragment
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitDepthNormalsPass.hlsl"
    ENDHLSL
}
```

7. 深度の書き込みことを確認します
![Desktop View](company_without/dof2.jpg){: width="642" height="425" .w-75 .normal}

8. 既存のコードを参考します (com.unity.render-pipelines.universal/Runtime/Passes/PostProcessPass.cs)
9. DoDepthOfFieldの肝心な部分を解説します

```csharp
void DoGaussianDepthOfField(CommandBuffer cmd, RTHandle source, RTHandle destination, Rect pixelRect, bool enableAlphaOutput)
{
	//四つのテクスチャを生成
    RenderingUtils.ReAllocateHandleIfNeeded(ref m_FullCoCTexture, GetCompatibleDescriptor(m_Descriptor.width, m_Descriptor.height, m_GaussianCoCFormat), FilterMode.Bilinear, TextureWrapMode.Clamp, name: "_FullCoCTexture");
    RenderingUtils.ReAllocateHandleIfNeeded(ref m_HalfCoCTexture, GetCompatibleDescriptor(wh, hh, m_GaussianCoCFormat), FilterMode.Bilinear, TextureWrapMode.Clamp, name: "_HalfCoCTexture");
    RenderingUtils.ReAllocateHandleIfNeeded(ref m_PingTexture, GetCompatibleDescriptor(wh, hh, GraphicsFormat.R16G16B16A16_SFloat), FilterMode.Bilinear, TextureWrapMode.Clamp, name: "_PingTexture");
    RenderingUtils.ReAllocateHandleIfNeeded(ref m_PongTexture, GetCompatibleDescriptor(wh, hh, GraphicsFormat.R16G16B16A16_SFloat), FilterMode.Bilinear, TextureWrapMode.Clamp, name: "_PongTexture");

    Blitter.BlitCameraTexture(cmd, source, m_FullCoCTexture, RenderBufferLoadAction.DontCare, RenderBufferStoreAction.Store, material, k_GaussianDoFPassComputeCoc);
    /*第一番目pass nearとfarの距離により、深度を0~1に挟みます
    half FragCoC(Varyings input) : SV_Target
    {
        UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);
        float2 uv = UnityStereoTransformScreenSpaceTex(input.texcoord);
        //最小值：等于相机的近裁剪面距离（如 Near=0.3，则最小返回 0.3）
        //最大值：等于相机的远裁剪面距离（如 Far=1000，则最大返回 1000）
        float depth = LOAD_TEXTURE2D_X(_CameraDepthTexture, _SourceSize.xy * uv).x;
        depth = LinearEyeDepth(depth, _ZBufferParams);
        half coc = (depth - FarStart) / (FarEnd - FarStart);
        return saturate(coc);
    }//这个函数 输出结果存入 _FullCoCTexture
    */		
	
    Blitter.BlitTexture(cmd, source, viewportScale, material, k_GaussianDoFPassDownscalePrefilter);
    /*第二番目pass 
    PrefilterOutput FragPrefilter(Varyings input)
    {
        UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);
        float2 uv = UnityStereoTransformScreenSpaceTex(input.texcoord);
    
        half farCoC = SAMPLE_TEXTURE2D_X(_FullCoCTexture, sampler_LinearClamp, uv).x;

        half4 color = SAMPLE_TEXTURE2D_X(_BlitTexture, sampler_LinearClamp, uv);
        color *= farCoC;
    
        PrefilterOutput o;
        o.coc   = farCoC;
        //alphaを除く
        //越远的的像素 越接近 白色
        //越近的像素 越接近 黑色
        o.color = half4(color.xyz, 0);
        return o;
    }
    */

    Blitter.BlitCameraTexture(cmd, m_PingTexture, m_PongTexture, RenderBufferLoadAction.DontCare, RenderBufferStoreAction.Store, material, k_GaussianDoFPassBlurH);
    Blitter.BlitCameraTexture(cmd, m_PongTexture, m_PingTexture, RenderBufferLoadAction.DontCare, RenderBufferStoreAction.Store, material, k_GaussianDoFPassBlurV);
    /*第三、四番目pass 二つ方向でぼかしを行います
    half4 FragBlurH(Varyings input) : SV_Target
    {
        return Blur(input, float2(1.0, 0.0), 1.0);
    }
    
    half4 FragBlurV(Varyings input) : SV_Target
    {
        return Blur(input, float2(0.0, 1.0), 0.0);
    }
    */

		
    Blitter.BlitCameraTexture(cmd, source, destination, RenderBufferLoadAction.DontCare, RenderBufferStoreAction.Store, material, k_GaussianDoFPassComposite);
    /*
    第五番目pass ぼかし結果と元画像を融合します
    half4 FragComposite(Varyings input) : SV_Target
    {
        UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);
        float2 uv = UnityStereoTransformScreenSpaceTex(input.texcoord);

        half4 baseColor = LOAD_TEXTURE2D_X(_BlitTexture, _SourceSize.xy * uv);
        half coc = LOAD_TEXTURE2D_X(_FullCoCTexture, _SourceSize.xy * uv).x;

        half4 farColor = SAMPLE_TEXTURE2D_X(_ColorTexture, sampler_LinearClamp, uv);
        half4 dstColor = 0.0;
        half dstAlpha = 1.0;

        UNITY_BRANCH
        if (coc > 0.0)
        {
            // Non-linear blend
            // "CryEngine 3 Graphics Gems" [Sousa13]
            //　重要な部分、この計算式いろんなことろで活用できそう
            half blend = sqrt(coc * TWO_PI);
            dstColor = farColor * saturate(blend);
            dstAlpha = saturate(1.0 - blend);
        }

        return half4(dstColor.rgb + baseColor.rgb * dstAlpha, 1.0);
    }
    */
}
```

## Postprocessで要望、仕様を実装するリスト

1. オブジェクト像を_CameraDepthTextureに書き込みます
2. カメラとの距離により、画面を三つ部分に分けます。近い部分、中部分、遠い部分です。
3. 近い部分と遠い部分をぼかしします。

## カメラとの距離により、画面を三つ部分に分けます。近い部分、中部分、遠い部分です。

1. 真っ黒部分をくっきりする
2. 近い部分と遠い部分をぼかしする

![Desktop View](company_without/dof3.png){: width="642" height="425" .w-75 .normal}

## 近いぼかし結果(boxBlur)
![Desktop View](company_without/dof5.jpg){: width="642" height="425" .w-75 .normal}

## 遠いぼかし結果 (noiseBlur)
![Desktop View](company_without/dof4.jpg){: width="642" height="425" .w-75 .normal}

## 融合した結果
![Desktop View](company_without/dof6.jpg){: width="642" height="425" .w-75 .normal}

## 自由に調整する
![Desktop View](company_without/dof7.jpg){: width="642" height="425" .w-75 .normal}

## コードダウンロード
[**RendererFeature.cs**][RenderFeature]
[**VolumeComponent.cs**][VolumeComponent]
[**CustomDepthOfField.shader**][Shader]

## 纏まり

1. 中央部分範囲は調整可能です
2. 近景と遠景は二つ異なるぼかし効果が効いている、美術センスを持っている人により、効果や表現は調整可能です。
3. 例えば、近景を別のぼかし効果を差し替えることができます。

## 参考

https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-features/custom-rendering-pass-workflow-in-urp.html


[RenderFeature]: https://github.com/zhangyile1991911/UnityShowcase/blob/main/Assets/PostprocessEffect/blur/CustomDepthOfFieldRendererFeature.cs
[VolumeComponent]: https://github.com/zhangyile1991911/UnityShowcase/blob/main/Assets/PostprocessEffect/blur/CustomDepthOfFieldVolumeComponent.cs
[Shader]: https://github.com/zhangyile1991911/UnityShowcase/blob/main/Assets/PostprocessEffect/blur/CustomDepthOfField.shader