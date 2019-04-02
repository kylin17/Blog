---
title: ADB备忘
date: 2019-04-02 11:46:06
categories:
- Android
tags: 
- Android
- ADB
---

挂载文件系统
# mount -o rw,remount /system
# mount -o ro,remount /system

隐藏系统UI
# pm disable com.android.systemui