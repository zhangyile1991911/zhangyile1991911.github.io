---
title: UnityでのUI管理システム設計
author: zhangyile
date: 2025-5-08 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/ui_framework
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 概要
UIページの管理はLRUというアルゴリズムで管理します。
LRUとは　The Least Recently Used Cache

### メリット
1. よく使われる画面を生成し直すことを避ける
2. インメモリ使用量を一定範囲内に制御できる
3. 長く使われない画面を自動的に廃棄する
4. 自動的にUI要素とスクリプトをバンディングする

### 例で説明する
- ユーザー動作　　　
    - ゲームが起動した時、ログイン画面が開いた
- ロジック側　　　　
    - スタックが空き枠があり、LoginWindowを入れた
![Desktop View](s1.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作　
    - そしてキャラクター一覧画面を開いてキャラー育成素材を確認したくなる
- ロジック側
    - スタックが空き枠があり、CharaWindowを入れた
![Desktop View](s2.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - バッグ画面を開いて持っているアイテムを確認してから、素材を購入しようと思ってショップ画面に遷移します。
- ロジック側
    - スタックが空き枠があり、BagWindowを入れた
![Desktop View](s3.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - ショップ画面を開いて購入しようアイテムを忘れてバッグ画面に戻ります
- ロジック側
    - スタックが空き枠があり、 ShopWindowを入れた
![Desktop View](s4.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - バッグ画面を開いた途端メールボックスにログイン報酬を思い出した
- ロジック側
    - BagWindowの使った回数に１とタイムスタンプを足す。回数だけ増加することに不親切なことが起きますかも。例えば繰り返してバッグ画面を開いた。バッグ画面の使った回数が大きな数字になってスタックに常駐になります。だからタイムスタンプ値を加えます。ずっと前に１００回以上開いた画面であっても廃棄されます
![Desktop View](s5.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - メール画面を開いて
- ロジック側
    - スタックが満載なので一番古いLoginWindowオブジェクトを削除した
![Desktop View](s6.jpg){: width="284" height="384" .w-75 .normal}

> 特別で常駐する画面[MainWindow]が他の処理で廃棄されないようにする

### 自動的にバンディング,UIページのコードを生成する
1. 画面を組み立てバンディングしたいオブジェクトを一定名前をつける
![Desktop View](s7.jpg){: width="713" height="405" .w-75 .normal}
2. 拡張したコマンドを利用する
![Desktop View](s8.jpg){: width="710" height="242" .w-75 .normal}
3. 生成するノードを枠に引き入れる
![Desktop View](s9.jpg){: width="714" height="405" .w-75 .normal}

## 生成Windowボタンを押す
1. プレハブが自動的に生成された
![Desktop View](s10.jpg){: width="588" height="200" .w-75 .normal}

2. 2.対応するコードが自動的にも生成された
![Desktop View](s11.jpg){: width="579" height="174" .w-75 .normal}

3. 生成されたコード

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using SuperScrollView;

/// <summary>
/// Auto Generated Class!!!
/// </summary>
[UI((int)UIEnum.LoginWindow,"Assets/GameRes/Prefabs/Windows/LoginWindow.prefab")]
public partial class LoginWindow : UIWindow
{
    public Image Img_bg;
    public TextMeshProUGUI Txt_Title;
    public TextMeshProUGUI Txt_UserName;
    public TextMeshProUGUI Txt_Passwd;
    public TMP_InputField Input_UserName;
    public TMP_InputField Input_Passwd;

    public override void Init(GameObject go)
    {
        uiGo = go;
        Img_bg = go.transform.Find("Img_bg").GetComponent<Image>();
        Txt_Title = go.transform.Find("Txt_Title").GetComponent<TextMeshProUGUI>();
        Txt_UserName = go.transform.Find("Txt_UserName").GetComponent<TextMeshProUGUI>();
        Txt_Passwd = go.transform.Find("Txt_Passwd").GetComponent<TextMeshProUGUI>();
        Input_UserName = go.transform.Find("Input_UserName").GetComponent<TMP_InputField>();
        Input_Passwd = go.transform.Find("Input_Passwd").GetComponent<TMP_InputField>();
    }
}
```