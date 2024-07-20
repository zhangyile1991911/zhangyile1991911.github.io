---
title: インディーゲーム会話ツール紹介
author: zhangyile
date: 2024-01-18 11:33:00 +0800
categories: [Indie Game]
tags: [Indie Game,Development]
comments: false
img_path: /assets/img/
image:
  path: yarnspinner/title.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

> このブログでは、今開発してるゲームで使用している会話ツールを紹介します。

## YarnSpinnerとは
プログラミング構造は簡単で拡張性が高い会話ツールです
公式ホームサイト:https://yarnspinner.dev 英語のみです


###　YarnSpinner選んだ理由
コードが分かりやすくて簡単にプロジェクトにインストールします。
会話内容のファイルをエンジンから抜き出して、他人に渡すことできますし。
例えば、会話を翻訳するのを専門家に渡す場合もありますから。でも自分のゲームまだ発表してない場合に役に立ちます。
発表しているゲームに他の言語追加するにも簡単にファイルだけを更新できます。
会話の中身で簡単なロジック制御ができます。

### インストール
インストール方法が三つあります。その中で一番簡単な方法はUnity Package Managerを通じてインストールする。

1. ![Desktop View](yarnspinner/install1.jpg)
2. ![Desktop View](yarnspinner/install2.png)
3. ![Desktop View](yarnspinner/install3.png)


## 手本を参考
### ツールを習うなら一番簡単な方法はsammpleを通じて勉強します。
「User Input and Yarn」の「import」ボタンをクリックしてsammpleが自動的に導入されます。
![Desktop View](yarnspinner/sample1.png)
### 以下は導入した結果
![Desktop View](yarnspinner/sample2.png)
### ファイル「inputSampleDialogue.yarn」をクリックして会話内容が出てきます。
![Desktop View](yarnspinner/sample3.png)

### このファイルは三つの重要な部分を分けて解読しましょう。
```
title: Start
```
この会話タイトルです。CSharpコードでこの会話を呼び出しための標識です。
```
<<declare $name = "">>
```
「name」という変数の名前を宣言し、値を格納するための記憶領域を確保させること。データ型は文字列です。
```
<<input name>>
```
「input」というメソッドを呼び、前に宣言した「name」の変数を代入する。

```
    void Start()
    {
        runner = FindObjectOfType<DialogueRunner>();
        //メソッドをスクリプトに注入します。
        runner.AddCommandHandler<string>("input", InputRenamer);
    }
    /// <summary>
    /// A blocking command that summons an input field, waits for input, and then stores the input into the variable.
    /// </summary>
    /// <param name="variable">The name of the yarn variable, without the $, to be inputted</param>
    public IEnumerator InputRenamer(string variable)
    {
        var accumulator = 0f;
        while (accumulator < fadeDuration)
        {
            var alpha = Mathf.Lerp(0, 1, accumulator/fadeDuration);
            inputGroup.alpha = alpha;
            accumulator += Time.deltaTime;
            //フーレンごとに透明度を一つずつ大きくします
            yield return null;
        }
        inputGroup.alpha = 1;
        //ユーザーのインプットのを待っています
        while (string.IsNullOrWhiteSpace(input))
        {
            yield return null;
        }
        //ユーザーがインプットした文字を変数に格納します
        runner.VariableStorage.SetValue($"${variable}", input);
        input = null;

        accumulator = 0;

        while (accumulator < fadeDuration)
        {
            var alpha = Mathf.Lerp(1, 0, accumulator/fadeDuration);
            //フレームごとに透明度を一つずつ小さくします
            inputGroup.alpha = alpha;
            accumulator += Time.deltaTime;
            yield return null;
        }
        inputGroup.alpha = 0;
    }
```


### 結果
![Desktop View](yarnspinner/result.gif)




