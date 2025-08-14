---
title: 最適化基本知識
author: zhangyile
date: 2025-8-14 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/titantitle.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 前提

> 直近、Timerの設計について色々考えています。
例えば　ゲーム内で最大限に1000個Timerがある場合はどうやって効率の高く実行できるかな

## 提案1

1. タイムスライスを利用する
2. 10アイテムがある配列を用意しておく
3. 50msの粒度で区切る

### 想定の流れ

```
    0 [0~50 ms]
    1 [50~100 ms]
    2 [100~150 ms]
    3 [150~200 ms]
    4 [250~300 ms]
    5 [300~350 ms]
    6 [350~400 ms]
    7 [400~450 ms]
    8 [450~500 ms]
    9 [500~550 ms]

    100ms後Callback Taskを追加する場合
    現在　処理番号　0
    floor(100ms/50ms粒度) - 1= 1
    0 + 1 = 1
    100ms後実行されるタスクが番号1スライスに入る

    Updateの処理
    現在　インデックスは０です。
    ０番スライス内のタスクをチェックして時間がついたタスクを処理する。
    そして処理したタスクを分類して一回のみタスクを削除し、
    ループ回数があるタスクを他のスライスに入れる。

    処理中スライスの残り時間があるか
    残り時間がない場合
        インデックスが次に進む
    残り時間がある場合
        何もしない
```

### 仮コード

```
struct TimerData
{
	public int Id;
	public float Remain;
	public float Interval;
	public int LoopTimes;
	public Action Callback;
}

List<List<TimerData>> timerList = new List<List<TimerData>>();
int timerIndex;
int AddTimer(float interval,int looptime,Action cb)
{
	int index = floor(interval%50) - 1 + timerIndex;
	Timer newTimer = new Timer(){};
	timerList[index].add(newTimer);
	return newTimer.Id;
}

void Update()
{
	List<TimerData> cur = timerList[timerIndex];
	for(int i = cur.length -1;i >= 0;i--)
	{
		one.Remain -= Time.deltaTime;
		if(one.Remain <= 0.0f)
			 one.Callback();
		else
			 continue;
			 
		cur.removeAt(i);
		one.LoopTimes -= 1;
		if(one.LoopTimes > 0 || one.LoopTimes == -1)
		{
			AddTimer()
		}			 
	}
}

```

### メリット

1. 毎回Update関数で全てのタスクをチェックする必要がありません。

### デメリット

1. 管理が複雑です


## 提案2 Simple is the best

1. 一つ500アイテム配列を用意しておく
2. 一フレーム毎　時間計算を行う

### コード

```
public class TimerModule
{
    struct TimerData
    {
        public int Id;
        public float Wait;
        public float Interval;
        public int LoopTimes;//-1 無限
        public Action Callback;
        public bool Occupied;
    }

    private List<Action> _actions = new List<Action>();
    private TimerData[] _TimerData = new TimerData[500];

    public void Init()
    {
        for (int i = 0; i < _TimerData.Length; i++)
        {
            _TimerData[i].Id = i;
            _TimerData[i].Wait = -1;
            _TimerData[i].Interval = -1;
            _TimerData[i].LoopTimes = -1;
            _TimerData[i].Callback = null;
            _TimerData[i].Occupied = false;
        }
    }

    public int AddTimer(float intervalMs,int loopTimes, Action callback)
    {
        if (loopTimes <= 0)
        {
            Debug.LogError($"AddTimer failed! loopTimes <= 0");
            return -1;    
        }
        if (intervalMs <= 0.0f)
        {
            Debug.LogError($"AddTimer failed! intervalMs <= 0.0f");
            return -1;    
        }
        for (int i = 0; i < _TimerData.Length; i++)
        {
            if(_TimerData[i].Occupied)
                continue;
            _TimerData[i].LoopTimes = loopTimes;
            _TimerData[i].Callback = callback;
            _TimerData[i].Interval = intervalMs;
            _TimerData[i].Occupied = true;
            _TimerData[i].Wait = intervalMs;
            return _TimerData[i].Id;
        }
        Debug.LogError($"AddTimer failed! out of range");
        return -1;
    }

    public void CancelTimer(int id)
    {
        if (id < 0 || id >= _TimerData.Length)
        {
            Debug.LogError($"CancelTimer failed! Invalid timer id: {id}");
            return;
        }

        _TimerData[id].Callback = null;
        _TimerData[id].Occupied = false;
        _TimerData[id].LoopTimes = -1;
        _TimerData[id].Interval = -1;
        _TimerData[id].Wait = -1;
    }
    
    public void Tick(float deltaTime)
    {
        deltaTime *= 1000f;
        _actions.Clear();
        for (int i = 0; i < _TimerData.Length; i++)
        {
            if(!_TimerData[i].Occupied)continue;
            
            float diff = _TimerData[i].Wait - deltaTime;
            if (diff > 0f)
            {
                _TimerData[i].Wait = diff;
                continue;
            }

            _actions.Add(_TimerData[i].Callback);
            if(_TimerData[i].LoopTimes == -1)continue;
            
            _TimerData[i].Wait = _TimerData[i].Interval + diff;
            _TimerData[i].LoopTimes--;
            if (_TimerData[i].LoopTimes == 0)
            {
                _TimerData[i].Callback = null;
                _TimerData[i].Occupied = false;
                _TimerData[i].LoopTimes = -1;
                _TimerData[i].Interval = -1;
                _TimerData[i].Wait = -1;
            }
        }
        doActions();
    }

    private void doActions()
    {
        foreach (var action in _actions)
        {
            action.Invoke();
        }
    }
}

```


1. Classを使うよりStructの方が良いです
2. Classの仕組みはインメモリの中で連続ではありませんので実際は参照を持ちます。
3. Structの仕組みは直接インメモリの中で連続配列でありますのでCPUが一気に全部データを読み取って処理し切れます。

```
struct Data { public int value; }
class RefData { public int value; }
void PerformanceTest()
{
    // Struct type cache hit rate
    Data[] structArray = new Data[1000000];

    // Class type cache hit rate
    RefData[] classArray = new RefData[1000000];
    for (int i = 0; i < classArray.Length; i++)
        classArray[i] = new RefData();

    Stopwatch sw = Stopwatch.StartNew();
    for (int i = 0; i < structArray.Length; i++)
        structArray[i].value++;
    sw.Stop();
    Debug.Log($"Struct Array: {sw.ElapsedMilliseconds} ms");

    sw.Restart();
    for (int i = 0; i < classArray.Length; i++)
        classArray[i].value++;
    sw.Stop();
    Debug.Log($"Class Array: {sw.ElapsedMilliseconds} ms");
}

//Struct Array: 1 ms
//Class Array: 4 ms
```

