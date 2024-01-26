---
title: ShaderGraphの練習1
author: zhangyile
date: 2023-12-11 13:02:00 +0800
categories: [Shader Graph]
tags: [Indie Game,Shader Graph]
comments: false
img_path: /assets/img/
image:
  path: shadergraph/dissolve.gif
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---


## 効果ご覧ください

![Desktop View](shadergraph/dissolve_shadergraph.png){: prepend: site.baseurl width="590" height="1026"}


## 説明

この部分の中に一番重要なノードはStepです。

```
ln >= Edge -> 1.0
ln < Edge -> 0

例えば　㏑=0.5
Edge > ln -> 0
StepのOut数値はSimpleNoiseから出る数値は0.5より大きいなら０になる


```