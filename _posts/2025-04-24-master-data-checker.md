---
title: マスタデータチェックツール
author: zhangyile
date: 2025-4-24 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 前提
> 毎回のメンテナンスする時にプランナー側がいつも小さいミスを犯してホットフィックスでミスを修正することが多いですから。なぜ事前にチェックしないかと思っていました。プランナーがマスタデータを弄た後にチェックできるツールを提案しました。

### 仕様の分析
1. Excelのシートの絡みが複雑すぎるから、プランナーと一々確認ことが不可能です。
2. シート間の紐付きに通用タイプが幾つあります。例 AシートのBカラムの値は必ずBシートのBカラムにあるはずとかカラムに重複値があるかなど通用チェックタイプです。
3. プランナーに自由に紐付きを弄らせるために、vue3.0を用いてウェブサイトを賄います。
4. 現在処理流れを影響されないようにするので、生成されてからチェックします

### 実装した機能
> 紐づき型
![Desktop View](company_without/master-checker-1.jpg)

> 重複チェック
![Desktop View](company_without/master-checker-2.jpg)

> ファイル名を自動補完
![Desktop View](company_without/master-checker-3.jpg)

> アイテムチェック
![Desktop View](company_without/master-checker-4.png)

> 絞り込み機能。もしこの中に一つExcelを集中しようとする場合もあります。Excel名を左上ボックスに入力したら、このExcelに関連チェックしか表示しないようになります。
![Desktop View](company_without/master-checker-5.jpg)