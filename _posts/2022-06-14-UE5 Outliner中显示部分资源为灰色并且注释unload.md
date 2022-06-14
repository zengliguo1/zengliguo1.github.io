---
title: UE5 Outliner中显示部分资源为灰色并且注释unload
date: 2022-06-14 22:17:00 +0800
categories: [Unreal5]
tags: [学习笔记, 问题解决]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    今天一打开UE5的场景，发现角色和小怪都不见了，一看Outliner，显示两个actor都变灰色了，并且注释着（unload），google了一下：

[![XIQrSP.jpg](https://s1.ax1x.com/2022/06/14/XIQrSP.jpg)](https://imgtu.com/i/XIQrSP)

    一开始还不知道这world partition是什么，又搜了下world partition，才知道是这个：

[![XIl074.jpg](https://s1.ax1x.com/2022/06/14/XIl074.jpg)](https://imgtu.com/i/XIl074)
    先左键长按拖拽选中全部
[![XIlwBF.jpg](https://s1.ax1x.com/2022/06/14/XIlwBF.jpg)](https://imgtu.com/i/XIlwBF)
    再右键，选第一个全加载就可以了
[![XIlUXT.jpg](https://s1.ax1x.com/2022/06/14/XIlUXT.jpg)](https://imgtu.com/i/XIlUXT)
