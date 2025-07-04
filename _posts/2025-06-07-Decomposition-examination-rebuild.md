---
title: 分解、考察、再構築
author: zhangyile
date: 2025-6-7 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/titantitle.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 前提
  直近　桜井正博さんの動画をみていますので、そこからヒントや仕事の姿勢、態度など知識を学んで活かしようと思っています。
  [動画](https://www.youtube.com/watch?v=hTNA84vJNEc)のC7の部分の知識を今回のタスクにとても役に立つと思うんです
## タスクの要望
- 奥行き感を出す

## 参考ゲーム
- タイタンスレイヤー
- 首なし騎士：異世界伝説

## 考察
![Desktop View](company_without/der1.png){: width="1398 × 650" height="1398 × 650" .w-75 .normal}
1. ２つゲームをスクリーンショットする
2. 並んで考察する
3. 同じところを見つける
  - 縦画面には背景が画面全体のどのぐらい高さを占めるという疑問の答えを見つけた。623/2400 = 26%
  - その先にどんな敵や景色など期待を持たせる雰囲気を作るのは一番重要なことだと思うんです
4. 異なるところを見つける
  - キャラクターの位置は違うです
5. 異なる理由は何故ですか
  - 進行方向が違うです.

### 進行方向
![Desktop View](company_without/titanslayermove.gif){: width="220" height="497" .w-75 .normal}
1. タイタンスレヤは一直線で奥行き,地面の巻きと噛み合わせることで表現を活かすため

![Desktop View](company_without/headlessmove.gif){: width="220" height="497" .w-75 .normal}
2. 首なし騎士はZ形で前へ進める

## 分解
### タイタンスレイヤー
このゲームは一番重要な表現は地面の巻き戻しです

> 想定の手法

- 地面のメッシュをカメラとの距離により、下に落ちる

![Desktop View](company_without/ground_image.png){: width="570" height="320" .w-75 .normal}

- メッシュの変更イメージ

![Desktop View](company_without/ground_mesh.png){: width="180" height="400" .w-75 .normal}

## 再現

```
Varyings vert (Attributes input)
{
    Varyings output;
    
    float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);

    float3 cameraWorldPos = _WorldSpaceCameraPos;
    float distanceToCamera = abs(positionWS.z - cameraWorldPos.z);
    output.blur = step(_CameraDist, distanceToCamera);
    
    float maxDownValue = 50;
    float down = smoothstep(_DownStart, (_DownStart + maxDownValue) * _DownAmplify , distanceToCamera);
    float downValue = clamp(0, maxDownValue, distanceToCamera - _DownStart);
    positionWS.y -= downValue * down;
    
    output.positionHCS = TransformWorldToHClip(positionWS);
    output.uv = TRANSFORM_TEX(input.uv, _MainTex);
    
    return output;
}
```

![Desktop View](company_without/ground_roll_mesh.png){: width="180" height="400" .w-75 .normal}

## マトメ
