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

### prefabの生成を集計する方法
バトルのメモリを軽量化する時に　どうやってメモリの使い情報を知れるか。
もしcc.instantiate呼び出すところをprefabの名前と回数を知ればいいと思います。
```
//BattleLoader.js
cleanup: function() {
    console.log("BattleLoader cleanup")
    //名前と回数を出力します
    Object.entries(this.counter).forEach(([key, value]) => {
        console.log('prefab ',key, ' 生成回数 ', value);
    });
    cc.instantiate = this.originalInstantiate;
}
 decoratorInstantiate: function(func, counter) {
        return function(...args) {
            const prefb = args[0];
            if (!counter.hasOwnProperty(prefb.name)) {
                counter[prefb.name] = 0;
            }
            const c = counter[prefb.name];
            counter[prefb.name] = c + 1;
            console.log(`Instantiate ${prefb.name}`);
            return func(prefb);
        };
},
onLoad: function() {
        //元の関数を保存
        this.originalInstantiate = cc.instantiate;
        this.counter = {};
        //書き換えます
        cc.instantiate = this.decoratorInstantiate(this.originalInstantiate, this.counter);
        cc.instantiate._clone = this.originalInstantiate._clone;
}
```
結果以下です
```
prefab  DamageNumberAnimation  生成回数  77
prefab  StateAssignEffect  生成回数  10
prefab  ComboEffect  生成回数  1
prefab  BattleTreasureBoxOffset  生成回数  1
prefab  PlayerUnit  生成回数  5
prefab  PlayerStatus  生成回数  5
prefab  CharacterCommandIcon  生成回数  5
prefab  AbnormalStateIcon  生成回数  224
prefab  SummonCommandIcon  生成回数  2
prefab  BattleMenuPopup  生成回数  1
prefab  BattleMessage  生成回数  1
prefab  EnemyUnit  生成回数  14
prefab  EnemyStatusMedium  生成回数  14
prefab  CharacterUnitStateEffectView  生成回数  5
prefab  EnemyUnitStateEffectView  生成回数  14
prefab  BattleStartAnimation  生成回数  1
prefab  ef_passiveskill  生成回数  1
prefab  TurnCountAnimation  生成回数  4
prefab  BattleActionIcon  生成回数  10
prefab  EnemyIcon  生成回数  5
prefab  CharacterIcon  生成回数  6
prefab  BattleTreasureBox  生成回数  3
prefab  EnemyUnitStateEffect  生成回数  12
prefab  WaveCountAnimation  生成回数  2
prefab  CharacterUnitStateEffect  生成回数  1
prefab  MissionNotificationItem  生成回数  1
prefab  BattleWinAnimation  生成回数  1
prefab  BattleEndResult  生成回数  1
prefab  BattleRewardIcon  生成回数  4
prefab  ItemIcon  生成回数  4
prefab  Count  生成回数  4
prefab  LevelUp  生成回数  1
prefab  BattleFriendRequestPopup  生成回数  1
prefab  CommonIcon  生成回数  1
```
上記見ると対応するprefabをすぐ分かる。
DamageNumberAnimationとAbnormalStateIconプール化するかどうか調査します。

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
