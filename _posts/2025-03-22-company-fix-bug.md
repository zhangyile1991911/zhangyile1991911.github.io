---
title: バグ調査「ノードを回収する前に破棄された」
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
> オブジェクトプールが内部ノードを破棄する際にエラーが出てきました

### 調査手順
1. ノードをプールに入れる時にログを出力してみます
結果により、ノードを入れる時はノーマル状態でしたことを確認しました
2. エラーメッセージ改めて確認した上で気付いたことが見つかりました
> ノードの_objectFlag が129になりました。129というはバイナリ形式で「 1000 0001」意味です。
```
var Destroyed = 1 << 0;
var Destroying = 1 << 7;
```
エンジンのソースコードを確認して破棄されたフラグという意味です。いわゆるこのノードもエンジンに廃棄されました。
3. 疑問が浮かんできた。どうしてプールが回収する前に破棄されましたか
4. ブレークポイントでコードを一時停止する
> ノードを名前につけてエンジンの破棄するコードに条件付きブレークポイントをつけてみます。
> 実行中のプログラムの状態を確認し、このノードは親ノードが破棄されるとともに破棄されました。
5. 5.どうしてノードは同時にプールに親ノードにありますか
> プールのソースコードを確認し
```
put: function (obj) {
    if (obj && this._pool.indexOf(obj) === -1) {
        // ソースコードから見るとノードを入れる前に
        // 親ノードから自分を取り除きました。
        obj.removeFromParent(false);
        var handler = this.poolHandlerComp ? obj.getComponent(this.poolHandlerComp) : null;
        if (handler && handler.unuse) {
            handler.unuse();
        }
        this._pool.push(obj);
    }
},
```
> 次は親ノードに追加するコードを確認し
```
addChild (child, zIndex, name) {
    if (CC_DEV && !cc.Node.isNode(child)) {
        return cc.errorID(1634, cc.js.getClassName(child));
    }
    cc.assertID(child, 1606);
    // 親ノードに追加する前にもちゃんとチェックすることも確認しました
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
6. ソースコードを確認した上でし正しく順番に扱えれば問題はないはずですが、でも上記の状態になることもあり得ないと言えません。
7. 例えばノードをプールに入れた後に親ノードに追加する仮説だったら、上記の状況になれるかもと考えてました。
8. 仮説を検証してみようと思います
9. ２つ場所でログを出力し、一つプールに入れる前に、も一つ親ノードに追加する前に
10. 問題はすぐ明きました。
11. 原因を説明します
> 下記のコードで簡単説明
```
const node = pool.get();
dosomething(node);
//もし上記のの関数内でノードも戻したら、下のコードを実行し続けばバグがおきます
Layer.addchild(node);

function dosomething(node)
{
    if (xxx)
    {//判断により、取り出したばかりノードをプールに戻すこともあります
        pool.push(node);
    }
    xxx
    xxx
}
```



