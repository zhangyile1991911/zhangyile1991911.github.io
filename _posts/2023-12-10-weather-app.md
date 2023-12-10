---
title: インディーゲームの日記
author: zhangyile
date: 2019-08-08 11:33:00 +0800
categories: [Indie Game]
tags: [Indie Game,Development]
comments: false
img_path: /assets/img/
image:
  path: sample.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 今日完了した機能

![Desktop View](gif/晴天播报切换1.gif){: prepend: site.baseurl width="295" height="513"}

### 説明
DOTweenとUniTaskいうライブラリを使い、画像を読み込みながら動きます

### コード
```
public async UniTask LoadSpriteAsync(string resPath,Action<Sprite> complete)
{
    var handler = YooAssets.LoadAssetAsync<Sprite>(resPath);
    _resHandles.Add(handler);
    await handler.ToUniTask();
    complete?.Invoke(handler.AssetObject as Sprite);
}

private async void switchSky(Weather weather)
{
    if (curWeather == weather) return;
    string sun_img_res = "";
    switch (weather)
    {
        case Weather.Sunshine:
            sun_img_res = "Assets/GameRes/Picture/UI/Phone/Weather/sky.png";
            break;
        case Weather.Rain:
            sun_img_res = "Assets/GameRes/Picture/UI/Phone/Weather/rainingSky.png";
            break;
    }

    Sprite loadSp = null;
    var s1 = ParentWindow.LoadSpriteAsync(
        sun_img_res,
        (sp) => loadSp = sp);
    //画像を読みながらアルファを変化する
    await UniTask.WhenAll(
        s1,
        Img_Sky.DOFade(0, 1f).ToUniTask());
    //二つのタスクが全部終わったら、またアルファを戻す
    Img_Sky.sprite = loadSp;
    Img_Sky.DOFade(1f, 1.0f);
}
```