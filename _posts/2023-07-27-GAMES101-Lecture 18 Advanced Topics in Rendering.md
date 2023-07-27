---
title: GAMES101-Lecture 18 Advanced Topics in Rendering
date: 2023-07-27 11:30:00 +0800
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

## Advanced Light Transport

### Unbiased light transport methods

* Unbiased无偏的，如果得到的期望是最后要的值一样，那就是无偏的

* biased有偏的

#### Bidirectional path tracing (BDPT)

* 摄像机打出一条光线后反射，光源打出一条光线，将两个端点连接起来

* 这个方法对于接受间接光的场景很好

* 缺点：
  
  * 实现困难
  
  * 运行慢

#### Metropolis light transport (MLT)

* A Markov Chain Monte Carlo (MCMC) application

* 当被积函数和概率密度函数的形状一致时，会比较好，这个方法可以生成和被积函数形状相同的pdf

* 这是一个局部的方法，给一个路径生成相似的路径

* 当光路比较复杂的时候，也就是越难的场景，这个方法的效果越好

* 缺点：
  
  * 不知道什么时候会收敛
  
  * 像素收敛速度不一样，所以有的图像会比较脏

### Biased light transport methods

#### Photon mapping(光子映射)

* 适合做Specular-Diffuse-Specular(SDS)

* 适合做generating caustics

* 第一步，从光源出发，发射光线直到打到diffuse的物体

* 第二步，从摄像机出发，发射光线直到打到diffuse的物体

* 第三步，计算局部的密度估计，光子分布越集中的地方越亮
  
  * 对于任何一个着色点，取它周围的最近的n个光子
  
  * 除以光子占的面积

* 很少的n，噪声就会很大

* 很大的n，就会变的模糊

* 这就是为什么就是有偏的的方法

* 因为我们该算一个点的密度，但实际算的是一个区域的密度，也就是说对密度的估计是个不对的估计

* 只有覆盖的面积无限小才会对，如果找同样的点数，那么发射的光子越多，越精确，也就是当发射足够多的光子时，就会得到正确的结果，所以这种方法就叫有偏的方法，也叫一致的方法consistent

* 对有偏的一个理解：
  
  * 有偏的 == 模糊的
  
  * 一致的 == 如果无限采样就不再模糊

#### Vertex connection and merging (VCM)

* 把双向路径追踪和光子映射结合起来

### Instant radiosity (VPL / many light methods)

## Advanced Appearance Modeling

## Non-Surface Models

## Surface Models
