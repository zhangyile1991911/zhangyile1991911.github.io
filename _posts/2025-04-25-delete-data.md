---
title: 定期的に不要なデータを削除するツール
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
> 毎回のメンテナンス時に、データベースのバックアップに大量の時間がかかっています。そのため、Pythonを使用してデータベース内の不要データを削除するツールを作成することを提案しました。

### 仕様の分析
1. 本番環境に影響を与えないように実行します。
2. 深夜帯に実行します。
3. Chatworkに進捗メッセージを出力します。
4. 終了時刻を設定できるようにします。例：Jenkinsで深夜に実行を開始し、午前10時までに処理を終了できるようにします。
5. CPU負荷を抑えるため、1回につき100件ずつデータを取得して削除します。
6. 削除処理が完了するたびにChatworkへ報告すると通知が多くなりすぎるため、報告間隔を調整できるようにします。例：10万件削除するごとに進捗を報告します。
7. 各削除処理の完了後に sleep(0.1) を実行します。これにより、CPUリソースを過度に占有しないようにします。
8. 削除対象のデータが数億件以上あるため、削除ツールを制御する管理プロセスを作成します。