---
title: ShaderGraphの練習3
author: zhangyile
date: 2024-01-01 22:02:00 +0800
categories: [Shader Graph]
tags: [Indie Game,Shader Graph]
comments: false
img_path: /assets/img/
image:
  path: shadergraph/flagwave/flagwave.gif
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

>　このブログでは、ShaderGraphを使用して簡単に旗が漂っている効果を紹介します。


### 段取り
まず、旗を移動するために、X軸を使います。
sin数値範囲は「-1～１」ですから。
左に寄ったり右に寄ったりさせるのはsin三角関数を利用してできるものです。

次、旗は下から上に渡って徐々に小さくなりますので、UV座標を利用すると思いついました。
UV座標範囲は「0～１」です。下の方によって0になるで上によって１になるんです。

### 実現
1.Xだけ取り出します
![Desktop View](shadergraph/flagwave/xasile.png)

2.移動距離を計算します
![Desktop View](shadergraph/flagwave/2.png)
SineTimeの数値と乱数を掛け合わせての結果をLerpメソッドに入力します。
UV座標のVによって上の方が移動距離が小さくなります。

3 最後
![Desktop View](shadergraph/flagwave/3.png)
前のLerp関数が出てきた結果とオブジェクトのX軸を掛け合わせて終わりました。


### 全部
![Desktop View](shadergraph/flagwave/4.png)
