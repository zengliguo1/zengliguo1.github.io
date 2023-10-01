---
title: GAMES104-Lecture 02 引擎架构分层与整体Pipeline
date: 2023-09-28 16:33:00 +0800
categories: [Computer Graphics, GAMES104]
tags: [图形学, 学习笔记]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: true
mermaid: true

---

## A Glance of Game Engine Layers

* 工具层：现代游戏引擎的界面，各种编辑器

* 功能层：让这个世界看得见动起来

* 资源层：各种数据，动画、模型、音乐，负责加载管理这些资源，为功能层提供弹药

* 核心层：功能层各个部分都会调用相同的很基础底层的代码，就像是工具箱、瑞士军刀

* 平台层：最终发步到用户的设备上，平台不一样、输入不一样（键盘、手柄、体感）

* 中间件和第三方库：会集成到上面的各个层中，有sdk可以直接编译进去；还有是工具，在引擎之外

[![pPqQ3iF.jpg](https://z1.ax1x.com/2023/09/30/pPqQ3iF.jpg)](https://imgse.com/i/pPqQ3iF)

### Resource

[![pPqQGRJ.jpg](https://z1.ax1x.com/2023/09/30/pPqQGRJ.jpg)](https://imgse.com/i/pPqQGRJ)

* 把资源的数据转换成引擎的高效数据：resource->assets；去掉无用信息

* 数据之间的关联reference是最重要的

* guid：资产的全局唯一识别号，相当于身份证

* 需要一个实时的资产管理器，handle系统

* 资产会根据进度进行加载和卸载，guid和handle就是解决这个问题
  
  * 延迟加载策略，比如材质从粗糙到细致的加载

### Function

[![pPqQ8G4.jpg](https://z1.ax1x.com/2023/09/30/pPqQ8G4.jpg)](https://imgse.com/i/pPqQ8G4)

* tick就是我们构建的世界的普朗克时间

* Tick函数
  
  * tickLogic：先把整个世界的物理规则算一遍，模拟出这个世界
  
  * tickRender：某个人看到的一副2d画面；会做裁剪、光照、阴影

* 多线程：前沿：job系统（多核并行，难点在于处理不同系统的前后关系）

[![pPqQUqx.jpg](https://z1.ax1x.com/2023/09/30/pPqQUqx.jpg)](https://imgse.com/i/pPqQUqx)

### Core

[![pPqQtMR.jpg](https://z1.ax1x.com/2023/09/30/pPqQtMR.jpg)](https://imgse.com/i/pPqQtMR)

* 数学库：数学效率
  
  * carmack's 1/sqrt(x)
  
  * sse：cpu并行运算向量

[![pPqQdZ6.jpg](https://z1.ax1x.com/2023/09/30/pPqQdZ6.jpg)](https://imgse.com/i/pPqQdZ6)

* 数据结构：在核心层做一套自己的数据结构，没有内存碎片

* 内存管理：开辟一大块内存自己管理，为追求最高的效率；cpu的缓存越大，取出数据效率越高；
  
  * 内存管理三大步骤：把数据放在一起，尽可能地顺序访问数据，读写的时候尽可能一起去读写

### Platform

[![pPqQJz9.jpg](https://z1.ax1x.com/2023/09/30/pPqQJz9.jpg)](https://imgse.com/i/pPqQJz9)

* 掩盖掉平台差异度

* Render Hardware Interface：重新定义一个api，把各个硬件sdk封装起来

[![pPqjQxA.jpg](https://z1.ax1x.com/2023/10/01/pPqjQxA.jpg)](https://imgse.com/i/pPqjQxA)

* 不同设备架构不同，如Ps还有scpu

### Tool

[![pPqjM2d.jpg](https://z1.ax1x.com/2023/10/01/pPqjM2d.jpg)](https://imgse.com/i/pPqjM2d)

* 真正的生产力，开发方式比较灵活，以开发效率优先

* 工具层是允许别人使用，工具层的代码量很大

* DCC(Digital Content Creation)：别人开发的资产生产工具（比如3dmax、maya、houdini）

* 从dcc到game engine的流程叫做asset condition pipeline，有导入器、导出器

### Layered Architecture

[![pPqQNs1.jpg](https://z1.ax1x.com/2023/09/30/pPqQNs1.jpg)](https://imgse.com/i/pPqQNs1)

* 封装，高层不知道底层的实现
* 底层与高层独立
+ 底层的代码不要轻易改动，顶层可以；只允许上调用下

## Mini Engine-Pilot

* ecs framework（支持多线程）

## Takeaways

* 引擎是分层架构的：
  
  * 平台层、核心层、资源层、功能层、工具层

* 越底层的代码越稳定质量越高，越高层的代码设计得越开放越灵活，能适应不同的游戏

* 游戏引擎的核心是tick函数
