---
title: GAMES101-Lecture 18 Advanced Topics in Rendering
date: 2023-07-27 16:05:00 +0800
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

[![pCv5Gss.jpg](https://s1.ax1x.com/2023/07/27/pCv5Gss.jpg)](https://imgse.com/i/pCv5Gss)

#### Metropolis light transport (MLT)

* A Markov Chain Monte Carlo (MCMC) application

* 当被积函数和概率密度函数的形状一致时，会比较好，这个方法可以生成和被积函数形状相同的pdf

* 这是一个局部的方法，给一个路径生成相似的路径

* 当光路比较复杂的时候，也就是越难的场景，这个方法的效果越好

* 缺点：
  
  * 不知道什么时候会收敛
  
  * 像素收敛速度不一样，所以有的图像会比较脏

[![pCv5lRg.md.jpg](https://s1.ax1x.com/2023/07/27/pCv5lRg.md.jpg)](https://imgse.com/i/pCv5lRg)

### Biased light transport methods

#### Photon mapping(光子映射)

* 适合做Specular-Diffuse-Specular(SDS)

* 适合做generating caustics

* 第一步，从光源出发，发射光线直到打到diffuse的物体

* 第二步，从摄像机出发，发射光线直到打到diffuse的物体

[![pCv51zQ.jpg](https://s1.ax1x.com/2023/07/27/pCv51zQ.jpg)](https://imgse.com/i/pCv51zQ)

* 第三步，计算局部的密度估计，光子分布越集中的地方越亮
  
  * 对于任何一个着色点，取它周围的最近的n个光子
  
  * 除以光子占的面积（求面积有各种各样的方法）

[![pCv58Mj.jpg](https://s1.ax1x.com/2023/07/27/pCv58Mj.jpg)](https://imgse.com/i/pCv58Mj)

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
* BDPT的子路径端点如果不能连线，就看能不能合并
* 使用光子映射的方式来处理合并附近的光子

[![pCv5JLn.jpg](https://s1.ax1x.com/2023/07/27/pCv5JLn.jpg)](https://imgse.com/i/pCv5JLn)

### Instant radiosity (VPL / many light methods)

* 实时辐射度算法

* 先发射light sub-path(BDPT)，并且假设每一条sub-path的端点是虚拟光源(VPL)virtual point light

* 也就是说使用直接光照的方法就能得到间接光照的效果

* 问题：
  
  * 有一些地方会莫名其妙的发光，和之前两个点的距离有关，当光源和物体表面的距离很接近就会除以一个接近0的数
  
  * VPL不能做Glossy的物体

[![pCv5tZq.jpg](https://s1.ax1x.com/2023/07/27/pCv5tZq.jpg)](https://imgse.com/i/pCv5tZq)

## Advanced Appearance Modeling

## Non-Surface Models

### Participanting media

* 散射介质

* 光打到云里，会打到小冰晶然后分散到各个地方去，当然，小冰晶也可能接受到其他反射过来的光

* 有一些光传着传着就没了，比如乌云

* 所以光线进入云之后会发生两件事情
  
  * 被吸收
  
  * 被散射：均匀散、反向散、前向散

[![pCvI8fO.jpg](https://s1.ax1x.com/2023/07/27/pCvI8fO.jpg)](https://imgse.com/i/pCvI8fO)

* 使用Phase Function相位函数来确定光线怎么散射（跟BRDF的key很像）
  
  * 随机选一个弹射方向、选一个发射距离（根据介质），在每一个shading point与光源连线

[![pCvI3tK.jpg](https://s1.ax1x.com/2023/07/27/pCvI3tK.jpg)](https://imgse.com/i/pCvI3tK)

### Hair/fur/fiber(BCSDF)

* 对于头发而言，要考虑光和一个曲线作用，而不是面

* 头发有两种高光
  
  * 一种无色的高光
  
  * 一种有色的高光

#### Kajiya-Kay Model

* 一根光线打到圆柱上，然后会散射到一个圆锥上，于此同时，也会向四面八方散射

* 但这个类似blinn-phong，并不真实

[![pCvI1k6.jpg](https://s1.ax1x.com/2023/07/27/pCvI1k6.jpg)](https://imgse.com/i/pCvI1k6)

#### Marschner Model

* 打到头发的光线，一部分会反射出去，另一部分会穿进头发里面去再穿出去（折射），叫TT，穿进头发后，打到头发内壁再反射回来，叫TRT

* 把头发当成玻璃的圆柱，由cuticle(表皮)和cortex(absorbs)组成，头发还有色素

[![pCvIQTx.jpg](https://s1.ax1x.com/2023/07/27/pCvIQTx.jpg)](https://imgse.com/i/pCvIQTx)

* 考虑了三种光线传播: R、TT、TRT

* 要多次散射，计算量很大

[![pCvIel4.jpg](https://s1.ax1x.com/2023/07/27/pCvIel4.jpg)](https://imgse.com/i/pCvIel4)

* 人物的头发模型渲染到动物皮毛是不对的

* 头发有三层结构
  
  * cuticle
  
  * cortex
  
  * medulla（髓质）：更容易发生散射（我理解成云一样的材质）

[![pCvIKmR.jpg](https://s1.ax1x.com/2023/07/27/pCvIKmR.jpg)](https://imgse.com/i/pCvIKmR)

* 人头发的medulla很少，但动物毛发的medulla很多，这就造成了不同

#### Double Cylinder Model(Linqi Yan发明的)

* 这个模型考虑了medulla

[![pCvIm6J.jpg](https://s1.ax1x.com/2023/07/27/pCvIm6J.jpg)](https://imgse.com/i/pCvIm6J)

* 除了有R、TT、TRT外，又新加了TTs(类似TT，但是更加散射)和TRTs(类似TRT，但更加散射)

[![pCvInX9.jpg](https://s1.ax1x.com/2023/07/27/pCvInX9.jpg)](https://imgse.com/i/pCvInX9)

[![pCvIM01.jpg](https://s1.ax1x.com/2023/07/27/pCvIM01.jpg)](https://imgse.com/i/pCvIM01)

### Granular material(颗粒材质)

* 一粒一粒的模型

* 运行时间很长

[![pCzrS5d.jpg](https://s1.ax1x.com/2023/07/28/pCzrS5d.jpg)](https://imgse.com/i/pCzrS5d)

## Surface Models

### Translucent material(BSSRDF)

* 半透明材质，但是不只是有光被吸收，光进入后还会发生散射，在其他地方射出

* 玉石、水母

#### Subsurface Scattering

* 次表面散射

* BSSRDF: 这次有四个参数，结合了BRDF后还加了入射点和出射点$S(x_i,w_i,x_0,w_0)$

* 因为不仅要考虑光对某个方向的贡献，还要考虑对哪个点做的贡献，所以既要对方向进行积分，还要对面积进行积分

L( $ x_ {0} $ , $ \omega _ {0} $ )= $ \int _ {A} $ $ \int _ {H^ {2}} $ S( $ x_ {i} $ , $ \omega _ {i} $ , $ x_ {o} $ , $ \omega _ {0} $ ) $ L_ {i} $ ( $ x_ {i} $ , $ \omega _ {i} $ ) $ \cos $ $ \theta _ {i} $ d $ \omega _ {i} $ dA

#### Dipole Approximation

* 当光打在半透明的物体上时，就好像在物体内部有一个光源，但一个还不够，在外面也需要一个，这就是这个方法的key

* 会有珠圆玉润的效果

[![pCzDxVe.jpg](https://s1.ax1x.com/2023/07/28/pCzDxVe.jpg)](https://imgse.com/i/pCzDxVe)

### Cloth

* 由缠绕的纤维组成

* 纤维缠绕会形成股ply，股再缠绕形成线yarn

[![pCzDzUH.jpg](https://s1.ax1x.com/2023/07/28/pCzDzUH.jpg)](https://imgse.com/i/pCzDzUH)

* BRDF

* Render as Participating Media
  
  * 把cloth当成体积，分成一个小小的格子

* Render as actual fibers

* 计算量很大

### Detailed material(non-statistical BRDF)

* 需要处理路径采样的问题

* 使用BRDF over a pixel，使用p-NDF(说实话，我的理解就是根据normal map来生成一个像素的pdf分布)

[![pCzDOKK.jpg](https://s1.ax1x.com/2023/07/28/pCzDOKK.jpg)](https://imgse.com/i/pCzDOKK)

[![pCzDH81.jpg](https://s1.ax1x.com/2023/07/28/pCzDH81.jpg)](https://imgse.com/i/pCzDH81)

[![pCzDbgx.jpg](https://s1.ax1x.com/2023/07/28/pCzDbgx.jpg)](https://imgse.com/i/pCzDbgx)

[![pCzDqv6.jpg](https://s1.ax1x.com/2023/07/28/pCzDqv6.jpg)](https://imgse.com/i/pCzDqv6)

* 波动光学（巨复杂，需要在负数域上积分）

[![pCzDjbD.jpg](https://s1.ax1x.com/2023/07/28/pCzDjbD.jpg)](https://imgse.com/i/pCzDjbD)

### Procedural appearance

* 噪声函数，使用噪声函数来程序化生成纹理、地形、水面

[![pCzDXDO.jpg](https://s1.ax1x.com/2023/07/28/pCzDXDO.jpg)](https://imgse.com/i/pCzDXDO)
