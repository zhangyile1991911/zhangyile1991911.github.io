---
title: UnityでのUI管理システム設計[改善中]
author: zhangyile
date: 2026-1-17 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 概要

ゲーム開発におけるUIの管理にいくつ問題がありますので
１．　UI表示順番管理
２.　UIが開く時に生成される、閉じる時に破棄される、頻繁にメモリを割り当てることを避けて処理負荷を減らす
３．UI要素をクラスの項目に紐つけるのが手間がかかる。毎回UI要素が変更されたら、紐付け直す必要がある
４.　実行中に内部変数をチェックしたり、バグが起きたときに調査したりしやすいためにツールを装備する
５．専用UI、アプリライフサイクルと同じように廃棄されずにUI要素がある　例えばホームページなどこのUI基盤を提出しました。
自分の経験を踏まえて
今まで経験してきた問題を省みながら見つけた方法の一つです。
ゲームUI開発によく遭った問題
１　表示の順番、どのオブジェクトが先や前に表示する
事前にBottom、Center、Middle、Top、いくつレイヤーを定義しておいて
表示順番が決まりました。

２　毎回UI要素が表示する時に生成される瞬間がCPUに処理負荷が高いので、
頻繁に生成と廃棄ことを避けるため、LRUというアルゴリズムを導入しました。
LRUとは　UI要素を格納するデータスペースが足りない時に、直近に使われた回数が一番少ない要素を廃棄する管理方法です。
３　UI要素を変数に紐つけることはとても手間がかかるし、UIの変更が発生したりすると、紐付け直すこともよくあります
以上の問題を解決するために、自動でコード生成することで時間を省けるようになる
４　実行中に内部変数を確認しやすく、バグが起きた時に調査しやすい　ツールを開発しました
内部変数は　UI要素現在の順位とか、前回開かれたタイミングなどデバグの役に立つ変数です
５　アプリのライフサイクルと同じような特殊のUI要素があれば、この要望を満たせるために、
IsPermentを提供しました
メモリに常駐する

## 仕組みの設計

- この基盤について礎はUIComponent,UIWindow,UIManager三つクラスです。
- UIComponentは最小単位のUI要素であります。
- UIWindowとは複数のUIComponentを搭載できます。
- 同じ名前UIWindowとUIComponentは一つのみです。
- UIManagerはLRUというアルゴリズムでUIWindowの管理を行います。
PS：LRUとはThe Least Recently Used Cache。


## メリット
1. よく使われる画面を生成し直すことを避ける
2. インメモリ使用量を一定範囲内に制御できる
3. 長く使われない画面を自動的に廃棄する
4. 自動的でスクリプトがUIに紐付く
5. 管理ツールで目安に内部変数や状態が見える

## 例で説明する
- ユーザー動作　　　
    - ゲームが起動した時、ログイン画面が開いた
- ロジック側　　　　
    - スタックが空き枠があり、LoginWindowを入れた
![Desktop View](ui_framework/s1.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作　
    - そしてキャラクター一覧画面を開いてキャラー育成素材を確認したくなる
- ロジック側
    - スタックが空き枠があり、CharaWindowを入れた
![Desktop View](ui_framework/s2.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - バッグ画面を開いて持っているアイテムを確認してから、素材を購入しようと思ってショップ画面に遷移します。
- ロジック側
    - スタックが空き枠があり、BagWindowを入れた
![Desktop View](ui_framework/s3.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - ショップ画面を開いて購入しようアイテムを忘れてバッグ画面に戻ります
- ロジック側
    - スタックが空き枠があり、 ShopWindowを入れた
![Desktop View](ui_framework/s4.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - バッグ画面を開いた途端メールボックスにログイン報酬を思い出した
- ロジック側
    - BagWindowの使った回数に１とタイムスタンプを足す。回数だけ増加することに不親切なことが起きますかも。例えばバッグ画面を繰り返して開いた。バッグ画面の使った回数が大きな数字になってスタックに常駐になります。だからタイムスタンプ値を加えます。ずっと前に１００回以上開いた画面であっても廃棄されます
![Desktop View](ui_framework/s5.jpg){: width="284" height="384" .w-75 .normal}

- ユーザー動作
    - メール画面を開いて
- ロジック側
    - スタックが満載なので一番古いLoginWindowオブジェクトを削除した
![Desktop View](ui_framework/s6.jpg){: width="284" height="384" .w-75 .normal}

> 特別で常駐する画面[MainWindow]が他の処理で廃棄されないようにする

## 自動的にバンディング,UIクラスのコードを生成する
1. 画面を組み立てバンディングしたいオブジェクトを一定名前をつける
![Desktop View](ui_framework/s7.jpg){: width="713" height="405" .w-75 .normal}
2. 拡張したコマンドを利用する
![Desktop View](ui_framework/s8.jpg){: width="710" height="242" .w-75 .normal}
3. 生成するノードを枠に引き入れる
![Desktop View](ui_framework/s9.jpg){: width="714" height="405" .w-75 .normal}

## 生成Windowボタンを押す
1. プレハブが自動的に生成された
![Desktop View](ui_framework/s10.jpg){: width="588" height="200" .w-75 .normal}

2. 2.対応するコードが自動的にも生成された
![Desktop View](ui_framework/s11.jpg){: width="579" height="174" .w-75 .normal}

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

## デバッグツール
1. 順位--LRUにより、現時点Windowの順位です。
2. 名前
3. component数
4. 所属階層--現時点windowはどの階層に所属しています
5. 活性化--現時点windowが表示されていますか
6. 使い回数
7. インメモリに永住するか

![Desktop View](ui_framework/tool.png){: width="579" height="174" .w-75 .normal}


## 基底クラスの拡張

1. 理由
    下記のような二つクラスが重複コードや機能を持つ場合だったら、共通のコードを抜き出して基底クラスを作る方が便利だと思います。

```csharp
class AComponent : UIComponent
{
	protected PlayableDirector rootPD;
	protected Animator rootAnimator;
	public override void OnCreate()
  {
      rootPD = uiTran.GetComponent<PlayableDirector>();
      rootAnimator = uiTran.GetComponent<Animator>();
  }
  public override void OnShow(UIOpenParam openParam)
  {//オブジェクトが表示時に何のアニメションを再生する
	  rootPD.Play();
  }
  
  public override void OnHide(UIOpenParam openParam)
  {//オブジェクトが非表示時に何のアニメションを再生する
	  rootPD.Play();
  }
}

class BComponent : UIComponent
{
	protected PlayableDirector rootPD;
	protected Animator rootAnimator;
	public override void OnCreate()
  {
      rootPD = uiTran.GetComponent<PlayableDirector>();
      rootAnimator = uiTran.GetComponent<Animator>();
  }
  public override void OnShow(UIOpenParam openParam)
  {
  }
  
  public override void OnHide(UIOpenParam openParam)
  {
  }
}
```

2. 共通コードを抜き出して基底クラスを作る

```csharp
class UIPDComponent : UIComponent
{
	protected PlayableDirector rootPD;
	protected Animator rootAnimator;
	public override void OnCreate()
  {
      rootPD = uiTran.GetComponent<PlayableDirector>();
      rootAnimator = uiTran.GetComponent<Animator>();
  }
  public override void OnShow(UIOpenParam openParam)
  {
  }
  
  public override void OnHide(UIOpenParam openParam)
  {
  }
}
```

3. 生成する前に基底クラスを選択する

![Desktop View](ui_framework/parentclasschoice.png){: width="579" height="174" .w-75 .normal}

4. 生成中に必須要件がチェックできる

```csharp
class UIPDComponent : UIComponent
{
	protected PlayableDirector rootPD;
	protected Animator rootAnimator;
	//子クラスが生成される時に必須要件をチェックする
	#if UNITY_EDITOR
    [UIComponentChecker]
    static bool CheckComponentExist(GameObject go)
    {
        var pd = go.GetComponent<PlayableDirector>();
        if(!pd)
        {
            UnityEngine.Debug.LogError($"UIPDComponent require PlayableDirector Component:{go.name}");
            return false;
        }
        var ar = go.GetComponent<Animator>();
        if(!ar)
        {
            UnityEngine.Debug.LogError($"UIPDComponent require Animator Component:{go.name}");
            return false;
        }
        return true;
    }
    #endif
}
```

## リポジトリ

[UIShowcase](https://github.com/zhangyile1991911/Unity_UIShowcase)