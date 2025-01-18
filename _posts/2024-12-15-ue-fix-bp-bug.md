---
title: Blueprintのバグの修復
author: zhangyile
date: 2024-12-15 10:55:00 +0800
categories: [Without]
tags: [Performance,Development,Alghorithm]
comments: false
img_path: /assets/img/
image:
  path: UnrealEngine/unrealenginetitle.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### Ghoul 攻击问题
> 现象: 
并不是 每次都会收到 攻击动画中插入的Animation Notify事件
> 定位过程  
1. 在PlayMontageAndWait前后插入日志，确保每次都被调用
2. 在WaitGameplayEvent前后插入日志，确保每次都被调用
> 初步定位
1. 确认PlayMontageAndWait每次都被调用
2. WaitGameplayEvent并不是每次都被调用
![Desktop View](UnrealEngine/bp_bug/1111.jpg)

通过等待Tag匹配
3. 打印当前MontageTag
4. 找到设置MontageTag地方确认
![Desktop View](UnrealEngine/bp_bug/2222.jpg)

5. 在GetRandomTaggedMontageFromArray中设置断点,发现被多次调用
6. 两次SET和PlayMontageAndWait会分别调用GetRandomTaggedMontagefromArray一次，因为这个函数是从数组中随机取出一个元素所以 播放的Montage后等待到的事件中Tag和SET时候Tag不一致，后续逻辑进行不下去