---
title: バグ調査「Android版動画再生」
author: zhangyile
date: 2025-3-14 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: UnrealEngine/isogashii_man.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 動画が再生する前にダウンロードしておき、スムーズに連続再生できるようにしました。
> 最初は直接にリモートURLを利用して動画を再生する手段をしました。
> そして連続再生する時に次の動画を切り替える瞬間に画面ブラックになってしまいました。
> この原因は次の動画を読み込んでいます。
> その現象を避けるように幾づ方法考えました。
1. 事前に動画ファイルをダウンロードする
2. 2つバッファを使って第一バッファを使っている時、第二バッファを次の動画を読み込みでおきます。
3. platformによって処理が違います。WEBでURL.createObjectURLを使って AndroidでgetWritableDirectory関数を呼んでフォルダを取得して書き込みます。
4. Android版はエンジン内にplatformの処理コードに問題があるのでローカルMP4ファイルを再生できませんでした。
> cocos/platform/android/java/src/org/cocos2dx/lib/Cocos2dxVideoView.java
```
//もしhostがない場合、getHost()の戻り値はnullです
Uri mVideoUri;
if (mVideoUri.getHost().length > 0) {
    mEtriever.setDataSource(mVideoUri.toString(), new HashMap<String,String>);
} else {
    mRetriever.setDataSource(mCocos2dxActivity.getContext(),mVideoUri);
}
```
> 正しい処理
```
Uri mVideoUri;
String host = mVideoUri.getHost();
if (host != null && host.length > 0) {
    mEtriever.setDataSource(mVideoUri.toString(), new HashMap<String,String>);
} else {
    mRetriever.setDataSource(mCocos2dxActivity.getContext(),mVideoUri);
}
```