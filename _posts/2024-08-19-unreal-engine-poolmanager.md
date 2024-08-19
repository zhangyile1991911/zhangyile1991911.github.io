---
title: UnrealEngine勉強日記
author: zhangyile
date: 2024-8-19 09:42:00 +0800
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
UGameInstanceSubsystem -> UWorldSubsystem


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
//由Blueprint提供实现，如果Blueprint没有提供对应的实现
//那就使用C++中 提供的实现
```

