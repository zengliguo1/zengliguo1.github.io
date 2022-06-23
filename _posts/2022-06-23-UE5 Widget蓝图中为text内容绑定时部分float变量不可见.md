---
title: UE5 Widget蓝图中为text内容绑定时部分float变量不可见
date: 2022-06-23 17:40:00 +0800
categories: [Unreal5, 问题总结]
tags: [学习笔记, 问题解决, Widget]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    在2dCharacter中创建了Lives变量（float），但在playerHud_WB中text的content上bind该变量时，发现找不到，明明可以看到像Melee Damage这样的float值，换成integer类型后就可以看到了，但这治标不治本，万一下次真的需要用到包含小数的变量呢？

    [![jPCs0I.jpg](https://s1.ax1x.com/2022/06/23/jPCs0I.jpg)](https://imgtu.com/i/jPCs0I)
    [![jPCrnA.jpg](https://s1.ax1x.com/2022/06/23/jPCrnA.jpg)](https://imgtu.com/i/jPCrnA)

    去论坛查了查，受到了点启发，点击create binding，将需要绑定的变量做成蓝图即可

    [![jPPVjH.jpg](https://s1.ax1x.com/2022/06/23/jPPVjH.jpg)](https://imgtu.com/i/jPPVjH)

    点击create binding，这里我已经创建过了，就是这个GetText_1

    [![jPPeud.jpg](https://s1.ax1x.com/2022/06/23/jPPeud.jpg)](https://imgtu.com/i/jPPeud)

    点击后会自动跳转到蓝图页面，将需要绑定的变量指向return value即可，其中自动给加了转换成double，看来这个变量需要变成double才能够被绑定，其实蛮奇怪的，因为有的float也不需要转换成double再去绑

    [![jPPD8U.jpg](https://s1.ax1x.com/2022/06/23/jPPD8U.jpg)](https://imgtu.com/i/jPPD8U)
