---
title: ShaderGraphの練習5
author: zhangyile
date: 2025-01-17 10:40:00 +0800
categories: [Shader Graph]
tags: [Indie Game,Shader Graph]
comments: false
img_path: /assets/img/
image:
  path: seebehind/result.gif
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

>　このブログでは、URPで遮られたものも見える効果を紹介します。

## 手順
1. 新規レンダリングレイヤー
2. レンダリングレイヤーにキャラーを配属
3. 新規RenderData
4. レンダリングのフローに処理を挿入

## 新規レンダリングレイヤー
![Desktop View](shadergraph/seebehind/seebehind_character.png)
追加する前にキャラー

![Desktop View](shadergraph/seebehind/seebehind_addlayer.png)
遮られるキャラーを特別に処理するために、新規レイヤーを追加します。



## レンダリングレイヤーにキャラーを配属
![Desktop View](shadergraph/seebehind/seebehind_putcharactertolayer.png)
キャラーを新規レイヤーに配属します

![Desktop View](shadergraph/seebehind/seebehind_disppear.png)
配属された、表示してたキャラーが表示しなくなりました。


## 新規RenderData
![Desktop View](shadergraph/seebehind/seebehind_addrenderfeature.png)
Edit--->Project Setting--->Graphics--->Scriptable Render Pipeline Settings
上記のパスで現在プロジェクトが使っているレンダリング配置を見つけます。


![Desktop View](shadergraph/seebehind/seebehind_addrenderobject.png)
パネルに2つRenderder Objectsを追加します。

## レンダリングのフローに処理を挿入
![Desktop View](shadergraph/seebehind/seebehind_addrenderobject2.png)
上記のスクショのように設置したら効果が出ます。
説明
一番目のRender Objectsの機能
Event　AfterRenderdingOpaques　この処理は不透明なもの処理した後に
Layer Mask 指定されたレイヤー
現在SeeBehindに配属されたオブジェクトをDepthと比べます。
オブジェクトはカメラとの距離が遠ければ遠いほどDepthの値が大きくなります。
現在のオブジェクトのDepthと既存Depthと比べて
もし現在オブジェクトのDepthが大きいなら、遮られましたということで
指定されたMaterialで表示します。

二番目のRender Objectsの機能
オブジェクトの遮らない部分を処理します。
