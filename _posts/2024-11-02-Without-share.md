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

## 前提
メモリ軽量化と高速化と最適化しようとしている時に一番重要なことは根拠、データに基づいてします。
自分の想像ではありません。
全ての改善は根拠がないといけません。

## デバッグ
### ノードに名前をつける
![Desktop View](company_without/have_name_node.png)
![Desktop View](company_without/no_name_node.png)
### prefabの生成を集計する方法
バトルのメモリを軽量化する時に　どうやってメモリの使い情報を知れるか。
もしcc.instantiate呼び出すところをprefabの名前と回数を知ればいいと思います。
しかし　既存コード内にcc.instantiateを新しい関数に書き換えるのはあり得ないです。
デコレータなら、簡単にできます。
#### デコレータ
デコレータとは、すでにある関数に処理の追加や変更を行う為の機能です。
```
//BattleLoader.js
cleanup: function() {
    console.log("BattleLoader cleanup")
    //名前と回数を出力します
    Object.entries(this.counter).forEach(([key, value]) => {
        console.log('prefab ',key, ' 生成された回数 ', value);
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
prefab  DamageNumberAnimation  生成された回数  77
prefab  StateAssignEffect  生成された回数  10
prefab  ComboEffect  生成された回数  1
prefab  BattleTreasureBoxOffset  生成された回数  1
prefab  PlayerUnit  生成された回数  5
prefab  PlayerStatus  生成された回数  5
prefab  CharacterCommandIcon  生成された回数  5
prefab  AbnormalStateIcon  生成された回数  224
prefab  SummonCommandIcon  生成された回数  2
prefab  BattleMenuPopup  生成された回数  1
prefab  BattleMessage  生成された回数  1
prefab  EnemyUnit  生成された回数  14
prefab  EnemyStatusMedium  生成された回数  14
prefab  CharacterUnitStateEffectView  生成された回数  5
prefab  EnemyUnitStateEffectView  生成された回数  14
prefab  BattleStartAnimation  生成された回数  1
prefab  ef_passiveskill  生成された回数  1
prefab  TurnCountAnimation  生成された回数  4
prefab  BattleActionIcon  生成された回数  10
prefab  EnemyIcon  生成された回数  5
prefab  CharacterIcon  生成された回数  6
prefab  BattleTreasureBox  生成された回数  3
prefab  EnemyUnitStateEffect  生成された回数  12
prefab  WaveCountAnimation  生成された回数  2
prefab  CharacterUnitStateEffect  生成された回数  1
prefab  MissionNotificationItem  生成された回数  1
prefab  BattleWinAnimation  生成された回数  1
prefab  BattleEndResult  生成された回数  1
prefab  BattleRewardIcon  生成された回数  4
prefab  ItemIcon  生成された回数  4
prefab  Count  生成された回数  4
prefab  LevelUp  生成された回数  1
prefab  BattleFriendRequestPopup  生成された回数  1
prefab  CommonIcon  生成された回数  1
```
上記見ると対応するprefabをすぐ分かる。
DamageNumberAnimationとAbnormalStateIconプール化するかどうか調査します。
#### AbnormalStateIconプール化した問題
ブール化しましたが　貸し出し数と回収された数全然合わないですから
チェックのために以下の関数を作成しました。
```
visitAllNodeFindName: function(node, path, findName, result) {
    if (node === null || node === undefined) {
        return;
    }

    for (let i = 0; i < node.children.length; i++) {
        const cur = node.children[i];
        if (cur.name.startsWith(findName)) {
            path.push(cur.name);
            result.push(path.join('_'));
            path.pop();
        }
        if (cur.childrenCount > 0) {
            path.push(`${cur.name}`);
            this.visitAllNodeFindName(cur, path, findName, result);
            path.pop();
        }
    }
}

let result = [];
this.visitAllNodeFindName(this.node, [], 'AbnormalStateIcon', result);
result.forEach(one => console.log('BattleLoader', 'cleanup', one));
```
#### Cleanupに関して
```
Component -> safeCleanup -> cleanup
Node -> safeDestroy
```

```
//このような使い方はイテレータの無効化になる
this.abnormalStateIconsRoot.children.forEach(child => {
    BattleViewManager.recyleAbnormalStateIcon(child);
});
```

## パフォーマンスチューニング(Performance Tuning)
### 普通処理
```
  console.time('create_pool_noasync');
  this._createDamageNumInstancePool();
  this._createEffectViewNodePool();
  this._createAbnormalStateIconPool();
  console.timeEnd('create_pool_noasync');
  //nothrottling create_pool_noasync: 37.01708984375 ms
  //nothrottling create_pool_noasync: 40.449951171875 ms
  //nothrottling create_pool_noasync: 38.274169921875 ms
  //4xslowdown create_pool_noasync: 95.0087890625 ms
```

### parralelの結果
```
console.time('create_pool_async');
  const task = [
      next => {
          this._createDamageNumInstancePool();
          next(null, 'damagenum');
      },
      next => {
          this._createEffectViewNodePool();
          next(null, 'effectView');
      },
      next => {
          this._createAbnormalStateIconPool();
          next(null, 'abnormalstate');
      }
  ];
  Async.parallel(task, () => {
      console.log("parallel complete");
      console.timeEnd('create_pool_async');
  });
  //nothrottling create_pool_parrallel: 40.882080078125 ms
  //nothrottling create_pool_parrallel: 42.236083984375 ms
  //nothrottling create_pool_parrallel: 34.85498046875 ms
  //4xslowdown _parrallel: 137.968017578125 ms
```

#### 読み込んでいるリソースを出力する 
エンジンのソースコード
```
function aggregateSpriteFrame() {
    const spriteFrameArr = [];
    cc.assetManager.assets.forEach((val, key) => {
        if (val instanceof cc.SpriteFrame) {
            const txt = val._texture;
            const obj = {
                spName: val._name,
                textureId: txt._id,
                textureName: txt._texture.name,
                ref: val._ref,
                nativeurl: txt._nativeUrl
            };
            if (txt._framebuffer) {
                if (txt._framebuffer._width) {
                    obj.fwidth = txt._framebuffer._width;
                }
                if (txt._framebuffer._height) {
                    obj.fheight = txt._framebuffer._height;
                }
            }
            spriteFrameArr.push(obj);
        } else if (val instanceof cc.SpriteAtlas) {
            console.log('SpriteAtlas', val._name);
        }
    });
    return spriteFrameArr;
}
```

### Chrome Devtools使い方
#### Performance
[1] devtoolsを開く
[2] recordを始める
![Desktop View](company_without/performance_step_1.png)
[3] 目立つところを探す
![Desktop View](company_without/performance_step_2.png)
[4] 問題ところを詳しく分析する
![Desktop View](company_without/performance_step_3.png)

#### Network

#### Memory


## 画像軽量化


### Docker
