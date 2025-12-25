---
title: IDisposableとUsingの活用
author: zhangyile
date: 2025-12-25 13:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 前提

例：ボタンを押すと画面が遷移する

### パタンA
```
class XXXWindow
{
	Button button;
    //遷移中のフラグ
	bool isIdle = true;

	void OnShow()
	{
	  //ボタンのイベントを監視する
     button.
     OnClickAsObservable.
	    TakeWhile((x) => isIdle).//遷移中だったら、何もしない
	    Subscribe((x) => {TransitionToOther()}).
	    RegisterTo(token);
	}
	
	async UniTask TransitionToOther()
	{
		 //入口と出口で値を正しく確保しなければなりません
		 //問題点
		 //1. 遷移中にバグもしくは不具合が出たら、isIdleはずっとtrueとなっている
		 isIdle = false;
		 await FadeOut();
		 await nextWindow.FadeIn();
		 isIdle = true;
	}
}
```

### パタンB
```
class XXXWindow
{
	Button button;
	bool isIdle = true;
	//IDisposableを継承してStructを作る
	internal readonly struct TransitionLock : IDisposable
	{
	    XXXWindow window;
	
	    public Transition(XXXWindow window)
	    {
	        this.window = window;
	        window.isIdle = false;
	    }
	    public void Dispose()
	    {
	        window.isIdle = true;
	    }
	}
	
	void OnShow()
	{
     button.
     OnClickAsObservable.
	    TakeWhile((x)=>isIdle).
	    Subscribe().
	    RegisterTo(token);
	}
	
	async UniTask TransitionToOther()
	{
		//自動解除
        using(new TransitionLock(this))
        {
            FadeOut();
            nextWindow.FadeIn();
        }
	}	
}
```

### 二つパタンを並べて見合わせる

```
async UniTask TransitionToOther()
{
	 isIdle = false;
     //もっとIf文やswitchが増えていくなら
	 if(current == Other)
	 {
		 isIdle = true;//忘れずに
		 return;
	 }
     else if(xxxx)
     {
        isIdle = true;//忘れずに
		return;
     }
     else if(yyyy)
     {
        isIdle = true;//忘れずに
		return;
     }
	 await FadeOut();
	 await nextWindow.FadeIn();

	 isIdle = true;
}

async UniTask TransitionToOther()
{
	//自動解除
	 using(new TransitionLock(this))
	 {
		 if(current == Other)return;//直接に戻す
         if(xxxx)return;
         if(yyyy)return;

		 FadeOut();
		 nextWindow.FadeIn();
	 }
}
```

### パタンBのメリット

    既存処理に複数条件判断を入れても遷移ロジックが正しく実行するようになります。

