---
title: Withoutエンジニア共有会
author: zhangyile
date: 2024-11-02 10:55:00 +0800
categories: [Without]
tags: [Performance,Development,Alghorithm]
comments: false
img_path: /assets/img/
image:
  path: company_without/mnsg.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## デバッグ
### ノードに名前をつける


## パフォーマンスチューニング(Performance Tuning)
### Chrome Devtools使い方
#### 読み込んだリソースを出力する 
エンジンのソースコード
cocos-engine/cocos2d/core/asset-manager/cache.js
```
    /**
     * !#en
     * Enumerate all content and invoke function
     * 
     * 
     * @method forEach
     * @param {Function} func - Function to be invoked
     * @param {*} func.val - The value 
     * @param {String} func.key - The corresponding key
     * 
     * @example
     * var cache = new Cache();
     * cache.forEach((val, key) => console.log(key));
     * 
     * @typescript
     * forEach(func: (val: T, key: string) => void): void
     */
    forEach (func) {
        for (var key in this._map) {
            func(this._map[key], key);
        }
    },
```

```
cc.assetManager.assets
//要素を順に走査する
cc.assetManager.assets.forEach(
  (val,key)=>console.log(key))

//キャッシュからTexture2Dを引き出す
let result =[];
cc.assetManager.assets.forEach(
  (val,key) => {
    if(val instanceof cc.Texture2D)
      result.push(val);
  }
);
result;

```
#### Network


#### Performance

#### Memory
##

## 画像軽量化

## メモリー軽量化

### プールを利用する

### Docker
