---
title: ObjectPoolについて考え
author: zhangyile
date: 2025-08-20 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/object_pool_tool.jpg
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
    private Queue<GameObject> _pool;
    ObjectPool()
    {
        _pool = new Queue<GameObject>(_capacity);
        MakeMore(50);       
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
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
            MakeMore(10);
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
    private Queue<GameObject> _pool;
    //貸したオブジェクトを記録する
    private Dictionary<int, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>();
        _history = new Dictionary<int, GameObject>();
        MakeMore(50);
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
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
           MakeMore(10);
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.GetInstanceID()] = gameObject;
        return gameObject;
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.GetInstanceID());
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

    int instanceID;
    void Start()
    {
        instanceID = GetInstanceID();
    }

    void OnDestroy()
    {
        //プールに報告します
        ParentPool?.ReportIllegalDestroy(instanceID);
    }
}

public class ObjectPool
{
    private Queue<GameObject> _pool;
    //貸したオブジェクトを記録する
    private Dictionary<int, GameObject> _history;
    ObjectPool()
    {
        _pool = new Queue<GameObject>();
        _history = new Dictionary<int, GameObject>();
        
        MakeMore(50);
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
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
            MakeMore(10);
        }

        GameObject gameObject = _pool.Dequeue();
        _history[gameObject.GetInstanceID()] = gameObject;
        return gameObject;
    }

    public void ReleaseObject(GameObject gameObject)
    {
        if (gameObject)
        {
            _history.Remove(gameObject.GetInstanceID());
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
    public void ReportIllegalDestroy(string name,int instanceId)
    {
        _history.Remove(name);
        Debug.LogWarning($"object name {name} has be illegally destroy!");
    }
}

```


## 借りたオブジェクトが自動的に回収される
> 以上の仮コードで返却、廃棄を含めて考えてみましたが、エンジニアが利用中に返却を意識しなきゃいけないから、面倒だと思いませんか。もしオブジェクトが自動的に返却してくれば、いかがですか


```
//Disposableを利用し　自動的に借りたオブジェクトをプールに返却する
//議論点　なぜここにclassを利用する　structを使わない
//class ObjectHandler h1 = pool.Get();
//class ObjectHandler h2 = h1;
//h2とh1は同じものです
//もしstructを使ったら
//struct ObjectHandler h1 = pool.Get();
//struct ObjectHandler h2 = h1;
//h2とh1は異なる
//h1.release();後
//h2.isRelease() == false

public class ObjectHandler : IDisposable
{
    public ObjectPool ParentPool { get; private set; }
    public GameObject Instance { get; private set; }

    public int InstanceId { get; private set;}
    public string name { get; private set;}
    private bool _hasReleased = false;
    public ObjectHandler(ObjectPool pool,GameObject instance)
    {
        ParentPool = pool;
        Instance = instance;
        InstanceId = instance.GetInstanceID();
        name = instance.name;
    }
    
    //手動で返却する
    public void Release()
    {
        ParentPool.ReleaseObject(this);
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
    public ObjectHandler handler;
    int instanceId;
    private void Start()
    {
        instanceId = GetInstanceID();
    }
    private void OnDestroy()
    {
        ParentPool?.ReportIllegalDestroy(handler);
    }
}

public class ObjectPool
{
    private Queue<GameObject> _pool;
    private Dictionary<int, GameObject> _history;//貸したオブジェクトを記録する
    ObjectPool()
    {
        _pool = new Queue<GameObject>();
        _history = new Dictionary<string, GameObject>();
        MakeMore(50);
    }

    private void MakeMore(int capacity)
    {
        for (int i = 0; i < capacity; i++)
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
        _history[gameObject.GetInstanceID()] = gameObject;
        ObjectHandler handler = new ObjectHandler(this, gameObject);
        return handler;
    }
    
    public void ReleaseObject(ObjectHandler handler)
    {
        if (handler)
        {
            _history.Remove(handler.Instance.GetInstanceID());
            _pool.Enqueue(handler.Instance);      
        }
    }

    public void LogLeakObject()
    {
        foreach (var pair in _history)
        {
            Debug.Log($"object name {pair.Key} leaked!");
        }
    }
    
    public void ReportIllegalDestroy(ObjectHandler handler)
    {
        _history.Remove(handler.InstanceID);
        Debug.LogWarning($"object name {handler.name} has be illegally destroy!");
    }
}
```

## 実に業務で発生したバグ
> 半年前に解決したバグをおもいだしました。[返却したオブジェクトを使い続ける](https://zhangyile1991911.github.io/posts/company-fix-bug/)


下記の最小再現コードでバグを顧みましょう

```

function dosomething(node)
{
    if (xxx)
    {//判断により、借りたばかりノードをプールに戻すこともあります
        pool.push(node);
    }
    xxx
    xxx
}

const node = pool.get();
dosomething(node);
//もし上記のの関数内でノードを返したとしても、下のコードを実行し続けるから　バグが起きます
Layer.addchild(node);


```

上記の場合で一旦オブジェクトが返却されたら　ObjectHandlerが持つオブジェクトを無効化するべきです。

```

public class ObjectHandler : IDisposable
{
    public void Release()
    {
        if(_hasReleased)return;
        ParentPool.ReleaseObject(this);
        // Instanceにヌールを代入する
        Instance = null;
        _hasReleased = true;
    }
}

```

終わる時にエラーが出るより、実行中に一刻早くエラーを出されたほうが良いと思います。

## オブジェクトが廃棄される時に自動で返却できません

```
internal class ObjectWatchDog : MonoBehaviour
{
    public ObjectPool ParentPool;
    private void OnDestroy()
    {//この関数が呼ばれた時は手遅れだった、回収できないようになっった
        ParentPool?.ReportIllegalDestroy(this.gameObject);
    }
}
```

## プールが自動的に拡張したり減削したりする

```
public class GameObjectPool
{
    public ObjectHandler Get()
    {
        //貸し出し際に 貸出回数を記録する
        //一定時間内で最大貸出数を記録する
        _currentBorrowed++;
        _maxBorrowed = Mathf.Max(_currentBorrowed, _maxBorrowed);
    }
    public void Release(ObjectHandler handler)
    {
        _currentBorrowed--;
    }

    public void LateUpdate()
    {
        //設定時間が経ったら
        _smoothedPeak = alpha * _maxBorrowed + (1.0f - alpha) * _smoothedPeak;
        int targetPoolNum = Mathf.CeilToInt(_smoothedPeak * 1.2f);
        ShrinkOrExpandTo(targetPoolNum);
    }

    private void ShrinkOrExpandTo(int targetPoolNum)
    {
        int current = _pool.Count;
        
        if (current == targetPoolNum)
            return;
        
        float changePercent = Mathf.Abs(targetPoolNum - current) / (float)current;
        //頻繁に操作を避けるため
        if (changePercent < 0.25f)
            return;

        if (targetPoolNum < current)
        {
            // shrink
            int toRemove = current - targetPoolNum;
            for (int i = 0; i < toRemove; i++)
            {
                var obj = _pool.Dequeue();
                Destroy(obj);
            }
            Debug.Log($"{name} shrink out of {toRemove}.");
        }
        else
        {
            // expand
            int toAdd = targetPoolNum - current;
            MakeMore(toAdd);
            Debug.Log($"{name} expand {toAdd}.");
        }
    }
}

```

## 貸出中オブジェクトリストを可視化、統計ツール

![Desktop View](company_without/object_pool_tool.jpg){: width="642" height="425" .w-75 .normal}

- 逸脱とはオブジェクトがプールから取り出されてから　親ノードを設定してないという状況です

## ツールリンク
具体的実現は下記リンクに記載されています。
[ObjectPoolShowcase](https://github.com/zhangyile1991911/ObjectPoolShowcase)

## 纏めリ
　以上のコードは簡単なところから、開発中でよく遭ったことを含めて対策を考えて少しずつ改善してきました。もちろん完璧ではありません。足りないものや考え不足ところがたくさんあるので、以下リストで次に進む方向を示します

### Todo list
- [ ] プールが自動的に拡張したり減削したりする
- [ ] 貸出中オブジェクトリストを可視化、統計ツールを作る
- [ ] ツールでテスト結果により、初期化時に適切なオブジェクト数を設定できるように