---
title: UnrealEngine勉強日記-2
author: zhangyile
date: 2024-8-18 09:42:00 +0800
categories: [Unreal Engine]
tags: [Unreal Engine,Development]
comments: false
img_path: /assets/img/
image:
  path: UnrealEngine/unrealenginetitle.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### UWorldsubsystem　と　UGameInstanceSubsystem

> UWorldSubsystem 提供一个和当前UWorld共同生命周期 UWorldと同じライフ・タイム

在Editor中会有多个UWorld
启动Editor->Editor
在Editor中启动游戏->PIE
打开Blueprint->EditorPreview
打开模型预览->EditorPreview

> UGameInstanceSubsystem 提供一个和游戏一样的生命周期 ゲームと同じライフ・タイム

> PIE生成順番
UGameInstanceSubsystem -> UWorldSubsystem -> GameMode


### FTickableGameObject
> 可以给UObject 提供 Tick方法，设计目的是 给 全局管理器 或者 单独游戏逻辑类

### GEditor
> GEditor->OnWorldDestroyed() 当UWorld销毁时候发出通知

### GetOuter()
> GetOuter() 获取当前对象，外部对象（父对象）


### BlueprintNativeEvent
```
UFUNCTION(BlueprintNativeEvent,BlueprintPure) 
void DoSomthing()
void DoSomthing_Implementation() 
//C++提供默认实现
//由Blueprint提供实现，如果Blueprint没有提供对应的实现那就使用C++中 提供的实现
//blueprintで実現し、もしblueprintで提供してないなら、C++の実現を利用する
```


### GameInstanceSubsystemの依存関係
```
//如果 ASubsystem依赖BSubsystem
//可以通过在ASubsystem中调用InitializeDependency 来初始化 BSubsystem
//ASubsystemがBSubsystemを依存するなら
//UASubsystem::InitializeでInitializeDependency関数を呼ぶことで
//BSubsystemの初期化を実行しておきます
void UASubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    bIsInitialized = true;
    UBSubsystem* first = Cast<UBSubsystem>(Collection.InitializeDependency(UBSubsystem::StaticClass()));
    UE_LOG(LogTemp,Log,TEXT("UASubsystem::Initialize %d"),first->bIsInitialized)
}
```

### UI最適化
```
Invalidation Box：适用于不需要频繁更新的静态 UI 场景。它通过缓存来减少不必要的重绘，适合优化大型、复杂但大部分时间不变的 UI。

Retainer Box：适用于需要更灵活控制更新频率的动态 UI 场景。它通过将内容渲染到 Render Target 上来减少绘制次数，并允许你设置更新频率，从而在性能与视觉效果之间找到平衡。
```
 