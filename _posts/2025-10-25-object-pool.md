---
title: ObjectPoolについて考え
author: zhangyile
date: 2025-08-20 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

> 今回に検討するObjectPool内のアイテムを利用する際に初期、リセット、廃棄など細かいことを無視してObjectPool自体に集中します

## 全ての物事は一番簡単なところからです
1. 貸出
2. 返す


```
public class ObjectPool
{
    private int _capacity = 50;
    private Queue<GameObject> _pool;
    ObjectPool()
    {
        _pool = new Queue<GameObject>(_capacity);
        for (int i = 0; i < _capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            _pool.Enqueue(one);
        }
    }

    public GameObject GetObject()
    {
        if (_pool.Count <= 0)
        {
            return null;
        }

        return _pool.Dequeue();
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _pool.Enqueue(gameObject);      
        }
    }
}

```

## 誰が借りたオブジェクトを返すことが忘れたら
> 以上の例で一番簡単なプールをつくりましたが、現実開発中でいつも不注意なエンジニアがいるので、借りたオブジェクトを返却せずことがよくあります。貸したオブジェクトを記録すれば、返却されないオブジェクトが分かるようになれば、助かると思うんです。


```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ObjectPool
{
    private int _capacity = 50;
    private Queue<GameObject> _pool;
    //貸したオブジェクトを記録する
    private Dictionary<string, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>(_capacity);
        _history = new Dictionary<string, GameObject>(_capacity);
        for (int i = 0; i < _capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            _pool.Enqueue(one);
        }
    }

    public GameObject GetObject()
    {
        if (_pool.Count <= 0)
        {
            return null;
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.name] = gameObject;
        return gameObject;
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.name);
            _pool.Enqueue(gameObject);      
        }
    }

    //返却せずオブジェクトを報告する
    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"object name{pair.Key} leaked!");
        }
    }
}

```


## 誰が借りたオブジェクトを返却せずに廃棄したら
> 上記のコードで貸し出したオブジェクトが記録されますが、もし貸出中でオブジェクトが違法廃棄されたら、どうします

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//GameObjectが廃棄されることを見張る
internal class ObjectWatchDog : MonoBehaviour
{
    public ObjectPool ParentPool;
    private void OnDestroy()
    {
        //プールに報告します
        ParentPool?.ReportIllegalDestroy(this.gameObject);
    }
}

public class ObjectPool
{
    private int _capacity = 50;
    private Queue<GameObject> _pool;
    //貸したオブジェクトを記録する
    private Dictionary<string, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>(_capacity);
        _history = new Dictionary<string, GameObject>(_capacity);
        for (int i = 0; i < _capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            //初期化時にComponentを付ける
            ObjectWatchDog dog = one.AddComponent<ObjectWatchDog>();
            dog.ParentPool = this;
            _pool.Enqueue(one);
        }
    }

    public GameObject GetObject()
    {
        if (_pool.Count <= 0)
        {
            return null;
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.name] = gameObject;
        return gameObject;
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.name);
            _pool.Enqueue(gameObject);      
        }
    }
    
    //返却せずオブジェクトを報告する
    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"object name {pair.Key} leaked!");
        }
    }
    
    //意外と廃棄される時に
    public void ReportIllegalDestroy(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.name);
            Debug.LogWarning($"object name {gameObject.name} has be illegally destroy!");
        }
    }
}

```


## 借りたオブジェクトが自動的に回収される
> 以上の仮コードで返却、廃棄を含めて考えてみましたが、エンジニアが利用中に返却を意識しなきゃいけないから、面倒だと思いませんか。もしオブジェクトが自動的に返却してくれば、いかがですか


```
//Disposableを利用し　自動的に借りたオブジェクトをプールに返却する
public class ObjectHandler : IDisposable
{
    public ObjectPool ParentPool { get; private set; }
    public GameObject Instance { get; private set; }

    private bool _hasReleased = false;
    public ObjectHandler(ObjectPool pool,GameObject instance)
    {
        ParentPool = pool;
        Instance = instance;
    }
    
    //手動で返却する
    public void Release()
    {
        ParentPool.ReleaseObject(Instance);
        _hasReleased = true;
    }
    
    public void Dispose()
    {
        //返却を忘れたら、廃棄される際に自動的に返却する
        if (!_hasReleased)
        {
            Release();    
        }
    }
}

//GameObjectが廃棄されることを見張る
internal class ObjectWatchDog : MonoBehaviour
{
    public ObjectPool ParentPool;
    private void OnDestroy()
    {
        ParentPool?.ReportIllegalDestroy(this.gameObject);
    }
}

public class ObjectPool
{
    private int _capacity = 50;
    private Queue<GameObject> _pool;
    private Dictionary<string, GameObject> _history;//貸したオブジェクトを記録する
    ObjectPool()
    {
        _pool = new Queue<GameObject>(_capacity);
        _history = new Dictionary<string, GameObject>(_capacity);
        for (int i = 0; i < _capacity; i++)
        {
            var one = new GameObject();
            one.name = $"xxx_ObjectPool_{i}";
            //初期化時にComponentを付ける
            ObjectWatchDog dog = one.AddComponent<ObjectWatchDog>();
            dog.ParentPool = this;
            _pool.Enqueue(one);
        }
    }

    public ObjectHandler GetObject()
    {
        if (_pool.Count <= 0)
        {
            return null;
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.name] = gameObject;
        ObjectHandler handler = new ObjectHandler(this, gameObject);
        return handler;
    }
    
    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.name);
            _pool.Enqueue(gameObject);      
        }
    }

    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"object name {pair.Key} leaked!");
        }
    }
    
    public void ReportIllegalDestroy(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.name);
            Debug.LogWarning($"object name {gameObject.name} has be illegally destroy!");
        }
    }
}
```

## 実に発生したバグ
> 半年前に解決したバグをおもいだしました。[返却したオブジェクトを使い続ける](https://zhangyile1991911.github.io/posts/company-fix-bug/)


下記の仮コードでバグを顧みましょう

```
const node = pool.get();
dosomething(node);
//もし上記のの関数内でノードを返したとしても、下のコードを実行し続けるから　バグが起きます
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

上記の場合で一旦オブジェクトが返却されたら　ObjectHandlerが持つオブジェクトを無効化するべきです。

```

public class ObjectHandler : IDisposable
{
    public void Release()
    {
        ParentPool.ReleaseObject(Instance);
        // Instanceにヌールを代入する
        Instance = null;
        _hasReleased = true;
    }
}

```

終わる時にエラーが出るより、実行中に一刻早くエラーを出したほうが良いと思います。

## 纏めリ
　以上のコードは簡単なところから、開発中でよく遭ったことを含めて対策を考えて少しずつ改善してきました。もちろん完璧ではありません。足りないものや考え不足ところがたくさんあるので、以下リストで次に進む方向を示します
1. プールが自動的に拡張したり減削したりする
2. 貸出中オブジェクトリストを可視化、統計ツール
3. ツールでテスト結果により、初期化時に適切なオブジェクト数を設定できるように