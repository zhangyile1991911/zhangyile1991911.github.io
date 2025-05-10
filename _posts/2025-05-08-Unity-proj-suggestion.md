---
title: Unityプロジェクトの基盤
author: zhangyile
date: 2025-5-08 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---
### 開発に便利化させるPlugin
## ゲーム内リストとグリッドプラジンplugin
[**ugui-super-scrollview**](https://assetstore.unity.com/packages/tools/gui/ugui-super-scrollview-86572)
# メリット
1. オブジェクト生成回数を減らすため
2. 色んなリストとグリッドの仕様やパタンが揃われている

## 敵やNPCのAI開発ツール
[**MonoBehaviourTree**](https://github.com/Qriva/MonoBehaviourTree)
# メリット
1. 拡張性が高く、カスタマイズしやすいです。
2. service,sequence,taskという基盤なコンポーネントが含まれています。
    
## マスターデーター
[**luban**](https://github.com/focus-creative-games/luban)
# メリット
1. カラムに数字で入力するより文字の方が読みやすいです。

| ItemId  | ItemType | Name    |
| :------ | :--------| ------: |
| 10001   | 硬貨      |  AAAA   |
| 20001   | AP回復薬  | CCCCCC  |
| 30001   | BP回復薬  | BDDDDDD |

2. ExcelでEnumを定義したら、Client側でもServer側でも統一されます。プログラマーがプランナーと揉めることを退げる。
3. 新入りメンバーにとってすぐ分かって力になれる気をさせるメリットもあります。
4. 主流な開発言語を生成できる
5. jsonやbinaryやprotobuff形式でデーターを生成できる
6. Excelでデーター構造を定義して,開発言語によりコードを生成できる
### 基盤システム

## UIの管理システム
[**UI管理のフレームワーク**](https://github.com/Qriva/MonoBehaviourTree)

## エンジン内でUpdateやFixedUpdateやTimerなど管理システム
# 理由
例えば、Enemyクラスの幾つ対象が生成された場合、対象ごとにupdateでロジック処理が行います。
EnemyA,EnemyB,EnemyC 三つの対象のupdate関数を呼ばれる順番がプラットフォームや機種により、一定ではりません。

- EditorでEnemyC::update—>EnemyB::update—→EnemyA::updateになるかもしれません
- WebでEnemyA::update—>EnemyB::update—→EnemyC::updateになるかもしれません。
- phoneでEnemyA::update—>EnemyC::update—→EnemyB::updateになるかもしれません。

不思議なバグが出るとか再現しにくくなりますから、UpdateServiceとTimerServiceという管理クラスを作ろうとします.
下記は想定コードです。

```csharp
class UpdateService
{
    SortedList<Func<float>> funcList;
    
    //priorityが大きな関数を優先されます.
    void Register(Func<float> func,int priority);
}

class Enemy
{
    void Tick()
    {
        Debug.Log(name);
    }
}
Enemy A;
Enemy B;
Enemy C;

UpdateService.Register(A.Tick,1);
UpdateService.Register(B.Tick,2);
UpdateService.Register(C.Tick,3);

//想定出力結果
C
B
A
//どんな環境やプラットホームで同じ順番で出力するのが可能です。
```


## Log出力管理システム
リスト
- システムにより、指定した色で出力できます。
- システムにより、システムの名をprefixとして出力できます。問題調査の時にフィルタで簡単に集中したい内容やLogをはみ出します。
- levelにより、設定したレベルが低いLogを出力されないです
```
[ItemSystem] 2025-05-05 13:33:00 xxx Id が足りない
[CharaSystem] 2025-05-05 13:33:00 xxx Id が存在しない
[GachaSystem] 2025-05-05 13:33:00 リクエストが発送失敗
```

## 状態の遷移や管理などステートマシンStateMachine
例えば ゲーム起動からログイン済みまでの流れ状態遷移

![Desktop View](company_without/statemachine.jpg){: width="600" height="500" .w-75 .normal}

仮コードで説明する

```csharp
interface IState
{
    void Enter();
    void Execute();
    void Exit();
}

class SimpleStateManager
{
        IState CurState;

        void ChangeTo(NewState)
        {
            CurState.Exit();
            //todo 遷移できるかチェック
            CurState = NewState;
            CurState.Enter();
        }

        void Update()
        {
            CurState.Execute();
        }
}

class LoadRes : IState
{
    SimpleStateManager Manager;
    void Enter()
    {
        
    }
    void Execute()
    {
        for(int i = 0; i < manifest.length;i++)
        {
            Asset.Load(manifest[i].resPath);
        }
        Manager.ChangeTo(new ConnectServer());
    }
    void Exit()
    {
    }
}

class ConnectServer : IState
{
    SimpleStateManager Manager;
    void Enter()
    {
        Connector.ConnectToServer();
    }
    void Execute()
    {
        if(Connector.IsConnected())
        {
                Manager.ChangeTo(new CheckResVersion());
        }
    }
    void Exit()
    {
    }
}

class CheckResVersion : IState
{
    void Enter()
    {
        if(!Connector.IsConnected())
        {
            Manager.ChangeTo(new ConnectServer());
        }
        Connector.RequestManifest(HandleResponse);
    }
    void Execute()
    {
        
        
    }
    void Exit()
    {
    }

    void HandleResponse(String Manifest)
    {
        if(Manifest.version == response.version)
        {
            Manager.ChangeTo(new EnterLogin());
        } 
        else
        {
            Manager.ChangeTo(new DownloadRes());
        }
    }
}

class DownloadRes : IState
{
    String[] DownloadResList;
    void Enter();
    {
        if(!Connector.IsConnected())
        {
            Manager.ChangeTo(new ConnectServer());
        }
    }

    void Execute()
    {
        for(int i = 0;i < DownloadResList.length;i++)
        {
            Connector.request(DownloadResList[i]);
        }
        Manager.ChangeTo(new EnterLogin());
    }

    void Exit()
    {
    }
}

class EnterLogin : IState
{
    void Enter()
    {
        SDK.Init();
    }
    void Execute()
    {
        if(PlayerPressButton())
        {
            SDK.Login(
                (success)=>{Manager.ChangeTo(new LoadGameScene())},
                (failed)=>{});
        }
    }
    void Exit()
    {
    }
}

class LoadGameScene : IState
{
    void Enter()
    {
        CharacterService.Init();
        BagService.Init();
        GachaService.Init();
    }
    void Execute()
    {
        if(CharacterService.IsInited()&&
        BagService.IsInited()&&
        GachaService.IsInited())
        {
            Manager.ChangeTo(new GameScene())
        }
    }
    void Exit()
    {
    }
}

class GameScene : IState
{
    void Enter()
    {
        UIService.OpenUI();
        
    }
    void Execute()
    {
    }
    void Exit()
    {
    }
}
```
## ObjectPool

## イベント事件を放送管理システム
Path Matchingというアリゴリズムを利用して放送システムを作成します

```csharp
//ユーザに関してPathを定義します
OutGame.User.Status.AP.Add
OutGame.User.Status.AP.Sub
OutGame.User.Status.BP.Add
OutGame.User.Status.BP.Sub

class Param
{
    String Path;
}

class UserStatusParam : Param
{
    int oldValue;
    int newValue;
}

class BroadcastService
{
    void RegisterPath(String Path,Func<Param> listener);
    void TriggerEvent(String Path,Param param);
}

// メッセージを受け取るクラス
class Test
{
    void ReceiveEvent(Param param)
    {
        Debug.Log(name);
    }
}
//想定書き方
Test t1;
BroadcastService.ReigsterPath("OutGame.User.Status.AP.*",t1.ReceiveEvent);
Test t2;
BroadcastService.ReigsterPath("OutGame.User.Status.AP.Add",t2.ReceiveEvent);
Test t3;
BroadcastService.ReigsterPath("OutGame.User.Status.AP.Sub",t3.ReceiveEvent);
Test t4;
BroadcastService.ReigsterPath("OutGame.User.Status.*.Add",t4.ReceiveEvent);

UserStatusParam usparam;
usparam.oldValue = 10;
usparam.newValue = 15;
BroadcastService.TriggerEvent("OutGame.User.Status.AP",usparam);
//想定出力
t1

UserStatusParam usparam;
usparam.oldValue = 10;
usparam.newValue = 15;
BroadcastService.TriggerEvent("OutGame.User.Status.AP.Add",usparam);
//想定出力
t1
t2
t4

UserStatusParam usparam;
usparam.oldValue = 15;
usparam.newValue = 5;
BroadcastService.TriggerEvent("OutGame.User.Status.AP.Sub",usparam);
//想定出力
t1
t3

UserStatusParam usparam;
usparam.oldValue = 5;
usparam.newValue = 15;
BroadcastService.TriggerEvent("OutGame.User.Status.BP.Add",usparam);
//想定出力
t4
```

## ネット状況監視システム
## ADVシステム
## 音や音楽再生管理システム