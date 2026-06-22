---
title: バグ調査「プール内のノードが回収される前に破棄された」
author: zhangyile
date: 2025-3-21 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 現象は
> オブジェクトプール内のオブジェクトを破棄する際にエラーが発生した

### 調査経由
1. オブジェクトをプールに返却する時にログを出力してみます
ログ上でオブジェクトが返却された時はノーマル状態であることを確認しました
2. エラーメッセージを改めて確認した上で気付いたことが見つかりました
> オブジェクトの_objectFlag変数 が129になりました。129というはバイナリ形式で「 1000 0001」という意味です。
> エンジンのソースコードで定義
```
var Destroyed = 1 << 0;
var Destroying = 1 << 7;
```
エンジンのソースコード上での定義は破棄されたフラグです。いわゆるこのオブジェクトがすでにエンジンに破壊されました。
3. 疑問が浮かんできた。オブジェクトがオブジェクトプールに返却する前に破壊されましたか
4. ブレークポイントで実行を一時停止する
> オブジェクトに名前をつけると破壊処理コードに条件付きブレークポイントをつける。
> 実行中にオブジェクトの状態を確認し、このオブジェクトは親オブジェクトに破棄されることが判明した
5. どうして同一オブジェクトは同時にオブジェクトプールと親オブジェクトの下に存在する
> オブジェクトプールのソースコードを解読する
```
put: function (obj) {
    if (obj && this._pool.indexOf(obj) === -1) {
        // ソースコード処理はオブジェクトをプールに投入する前に
        // 現在の親オブジェクトから自分を取り除きました。
        obj.removeFromParent(false);
        var handler = this.poolHandlerComp ? obj.getComponent(this.poolHandlerComp) : null;
        if (handler && handler.unuse) {
            handler.unuse();
        }
        this._pool.push(obj);
    }
},
```
> 次はオブジェクト下に追加するコードを解読する
```
addChild (child, zIndex, name) {
    if (CC_DEV && !cc.Node.isNode(child)) {
        return cc.errorID(1634, cc.js.getClassName(child));
    }
    cc.assertID(child, 1606);
    // オブジェクト下に追加する前にもちゃんとチェックすることも確認しました
    cc.assertID(child._parent === null, 1605);
    child.parent = this;
    if (zIndex !== undefined) {
        child.zIndex = zIndex;
    }
    if (name !== undefined) {
        child.name = name;
    }
},
```
6. ソースコードを解読した上でし正しく順番に扱えれば問題はないはずですが、上記の状態になることもあり得ないと言えません。
7. 例えばオブジェクトをプールに返却後に親オブジェクト下に追加する仮説が成立したら、上記の状況になれるかもと考えてました。
8. 仮説を検証してみようと思います
9. 一つはオブジェクトプールに返却する前に、も一つは親オブジェクトに追加する前に、この２つ場所でログを出力する
10. 問題はすぐ判明しました。
11. 原因を説明します
> 下記は最小再現コードで簡単説明します

```

function dosomething(node)
{
    if (xxx)
    {//判断により、借りたばかりオブジェクトをプールに戻すこともあります
        pool.push(node);
    }
    xxx
    xxx
}

const node = pool.get();
dosomething(node);
//もし上記のの関数内でオブジェクトが返却されたとしても、下のコードを実行し続けるのでバグが発生します
Layer.addchild(node);


```