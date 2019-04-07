---
title: iOS Animation
date: 2017-10-30 23:02:35
tags:
---
一些常用的动画特效
<!--more-->

## 弹簧果冻动画效果
```swift
// 按钮果冻效果
let animation = CAKeyframeAnimation(keyPath: "transform.scale")
animation.values = [1.2, 0.8, 1.1, 0.9, 1.0]
animation.duration = 0.25
checkBtn.layer.add(animation, forKey: "")
```
