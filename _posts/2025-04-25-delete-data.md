---
title: 定期的に要らないデータを削除ツール
author: zhangyile
date: 2025-4-25 09:42:00 +0800
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
> 毎回のメンテナンスする時にデータベースのデータをバックアップすることでたくさん時間かかってますから。pythonでデータベースの中に要らないデータ削除という提案しました。

### 仕様の分析
1. 本番に影響を持たさないように
2. 深夜から実行します
3. chatworkに進捗メッセージを報告できます
4. 終了時間を設定できます。例：jenkinsで深夜に実行が始まり、午前十時まで終わると言うことが出来るように
5. CPUに負荷が掛からないように毎回100件データを取って削除します。
6. 毎回削除作業が終わるとchatworkに報告するとなると煩わしくなりがちですから、報告件数というパラメタを提供します。例えば十万件ごとに報告します
7. 毎回削除作業が終わた後にsleep(0.1)をさせます。止まらないとCPU利用率を全部占められる可能性があります。
8. 削除件数が具体的に分からないので、削除ツールを回す管理プロセスを作ります。