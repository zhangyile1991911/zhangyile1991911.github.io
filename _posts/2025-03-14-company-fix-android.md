---
title: バグ調査「Android版動画再生」
author: zhangyile
date: 2025-3-14 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 動画を滑らかに連続して再生するため、動画をダウンロードしてから再生する
> 最初はリモートURLを直接アクセスして動画を再生する手段となっていました。
> そして次の動画を再生する際に画面が一時にブラックになってしまいました。
> この原因は次の動画はまだロード中です。
> その問題を改善するため、幾づ方法考えました。
1. 事前に動画ファイルをダウンロードする
2. 2つバッファを用意して1番目バッファが使用中時、2番目バッファが次の動画を読み込みでおきます。
3. platformによってメソッドが違います。WEBプラットフォームはURL.createObjectURL、AndroidはgetWritableDirectory、この二つのメソッドでフォルダを取得できます。
4. Android版のソースコードにはバグがあるため、ダウンロード動画ファイルを再生できませんでした。

> cocos/platform/android/java/src/org/cocos2dx/lib/Cocos2dxVideoView.java
```
//hostが未設定場合、getHost()の戻り値はnullです
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