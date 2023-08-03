---
title: GAMES101-Lecture 21 Animation
date: 2023-08-03 16:08:00 +0800
categories: [Computer Graphics, GAMES101]
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

## Animation

* 是模型的拓展

* 电影：24 frames per second

* 视频：整体上都是30fps

* 虚拟现实：90fps（不晕的话）

## History

* 早期动画，把动画画在一个圆盘上

* 起初动画是为了科学研究，但渐渐发现人们喜欢看动态的东西，所以就向娱乐化发展

* 1963年就有计算机的动画了，1972就有人脸的动画了，1993侏罗纪公园，1995玩具总动员......

## Keyframe animation

* 关键帧动画

* 每一个帧有若干重要的点，这些重要的点在其他帧长什么样子，重要点之间的关系找到，然后通过插值的方式找出中间的部分来

* 希望有一种方式控制插值的效果（比如动作渐变），因此，曲线、样条应运而生

* 动画和几何是很有关联的

## Physical simulation

* 使用牛顿定律

* 模拟的好的话，不会发生违反物理的原理（至少不会穿模

### Mass Spring System

[![pPifGYn.jpg](https://s1.ax1x.com/2023/08/03/pPifGYn.jpg)](https://imgse.com/i/pPifGYn)

* 质点弹簧系统

* x表示位置

* $\dot{x} = v$ 表示一阶导数 

* $\ddot{x} = v $ 表示二阶导数

* Simple motion damping简单运动阻尼
  
  * 这个并没有考虑弹簧内部的损耗

* Internal Damping for Spring
  
  * 前面的$\frac{b-a}{||b-a||}$是用来算后面$\dot{b}-\dot{a}$的方向的，也就是取正号还是负号，但是这个力必须得沿着速度方向投影，才算真正能作用到弹簧上的力，所以后面要点乘一个$\frac{b-a}{||b-a||}$

[![pPifJWq.jpg](https://s1.ax1x.com/2023/08/03/pPifJWq.jpg)](https://imgse.com/i/pPifJWq)

### Structures from Springs

[![pPif8Fs.jpg](https://s1.ax1x.com/2023/08/03/pPif8Fs.jpg)](https://imgse.com/i/pPif8Fs)

* 简单的弹簧sheet没有抗切变的力，也没有抗弯曲的力，所以不能正确模拟布
  
  * 所以在每一格中加一条斜线，但是又不对称了，有了各向异性
  
  * 再加一条斜线，但仍然不能正确抗弯曲
  
  * 任何一个点和相隔的一个点连线，加一点点非平面的弯折

[![pPif1oj.jpg](https://s1.ax1x.com/2023/08/03/pPif1oj.jpg)](https://imgse.com/i/pPif1oj)

### Aside: FEM(Finite Element Method) Instead of Springs

* 有限元方法

[![pPifnQf.jpg](https://s1.ax1x.com/2023/08/03/pPifnQf.jpg)](https://imgse.com/i/pPifnQf)

## Particle System

* 根据需要创建新的粒子

* 定义每个粒子受到的力
  
  * 重力、电子力、斥力、弹力、牵引力、摩擦力、空气阻力、粘滞力
  
  * 碰撞

* 更新粒子的位置和速度

* 移除死亡的粒子

* 渲染粒子

* 都是先模拟再渲染，水也可以是粒子模型

[![pPifmSP.jpg](https://s1.ax1x.com/2023/08/03/pPifmSP.jpg)](https://imgse.com/i/pPifmSP)

### Simulated Flocking as an ODE

* 模拟鸟群

[![pPiflwQ.jpg](https://s1.ax1x.com/2023/08/03/pPiflwQ.jpg)](https://imgse.com/i/pPiflwQ)

## Kinematics

[![pPifQeg.jpg](https://s1.ax1x.com/2023/08/03/pPifQeg.jpg)](https://imgse.com/i/pPifQeg)

* 运动学

* 骨骼系统：定义不同的关节

* 关节类型：
  
  * Pin(一维旋转)
  
  * Ball(二维旋转)
  
  * Prismatic joint(可以移动)

* 定义好连接方式，就可以计算运动位置

[![pPifuy8.jpg](https://s1.ax1x.com/2023/08/03/pPifuy8.jpg)](https://imgse.com/i/pPifuy8)

### Kinematics Pros

* 艺术家喜欢拖拽

### Inverse Kinematics

* 逆运动学

* 计算复杂，而且存在多解的问题

[![pPifKOS.jpg](https://s1.ax1x.com/2023/08/03/pPifKOS.jpg)](https://imgse.com/i/pPifKOS)

## Rigging

* 对于木偶的一个操作，吊索

* 软选取、蒙皮

[![pPFSvHH.jpg](https://s1.ax1x.com/2023/08/03/pPFSvHH.jpg)](https://imgse.com/i/pPFSvHH)

### Blend Shape

* 定义若干不同的状态，中间是插值出来的

## Motion Capture

* 动作捕捉，加速获取控制点数据

* 缺点：
  
  * 准备麻烦
  
  * 捕捉出的动作不一定符合要求
  
  * 数据不一定好
  
  * 摄像机成本
  
  * 身体遮挡

* 有多种动捕方法
  
  * Optical
  
  * Magnetic
  
  * Mechanical

[![pPFSzEd.jpg](https://s1.ax1x.com/2023/08/03/pPFSzEd.jpg)](https://imgse.com/i/pPFSzEd)

## Challenges of Facial Animation

* 通过真实的动画生成方式产生了恐怖谷效应

## The Production Pipeline
