---
title: 2つ異なるぼかし表現を実装する
author: zhangyile
date: 2025-5-20 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/two_different_blur_result.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 前提

    > 近景と遠景のぼかし表現を別にして欲しいです。近景のぼかし程度は一定です。遠景のぼかし程度はゴールに近づけるとともにはっきり見えていきます。

## 仮想定

1. 各オブジェクトがぼかし計算を行うより描画の最終段階で一括で処理したほうが効率が高いだと思うんです
2. 近,中,遠、画面を3つ部分に分けます
3. 近景をnoiseBlurでぼかしします。中央部分はないもしません。遠景をboxBlurでぼかしします。

## PostProcessを用いて実装してみる

1. 新しいPostProcessを追加する
