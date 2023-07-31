---
title: GAMES101-Lecture 19 Cameras, Lenses and Light Fields
date: 2023-07-28 16:05:00 +0800
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

## Pinhole Image Formation

* 针孔相机没有深度，每个地方都是清晰的，看不到虚化的地方

* 光线追踪的时候用的就是针孔摄像机的原理，所以就没有景深的效果

## Field of View(FOV) 视场

### Effect of Focal Length on FOV

* fov和传感器大小和焦距有关，但默认传感器是有一个固定的大小，也就是说焦距是按照35mm的胶片(film)（也可以说是传感器）去定义的，其实传感器可能大小不同，但焦距都是按35mm的胶片去定义的

* 更长的焦距对应更小更远的视场

[![pPpxKEV.jpg](https://s1.ax1x.com/2023/07/31/pPpxKEV.jpg)](https://imgse.com/i/pPpxKEV)

### Effect of Sensor Size on FOV

* 更大的传感器，对应更大更近的视场

* 传感器sensor和胶片film区分：
  
  * sensor吸收irradiance
  
  * film用来决定最后存成什么样的格式

[![pPpxmBq.jpg](https://s1.ax1x.com/2023/07/31/pPpxmBq.jpg)](https://imgse.com/i/pPpxmBq)

## Exposure

* H = T \* E

* Exposure(H)= time(T) \* irradiance(E)

* 快门控制进光时间

* 光圈大小和焦距会影响到irradiance

[![pPpxMNT.jpg](https://s1.ax1x.com/2023/07/31/pPpxMNT.jpg)](https://imgse.com/i/pPpxMNT)

### Exposure Controls

* Aperture size光圈大小：改变光圈大小来改变f-stop

* Shutter Speed快门速度：改变快门速度来改变感光时间

* ISO gain感光度：后期处理，增加最后图片的曝光，可以调整硬件也可以调整软件，简单放大信号会同时放大噪声

* 为什么会有噪声，感光时间不足，光子就不足，就会产生noise

### ISO

* 线性增加

### F-Number(F-Stop)

* 不正式的理解：F数是光圈直径的倒数

### Side Effect Shutter Speed

* 会产生运动模糊

* Rolling shutter：对于超级高速的东西产生扭曲

### Constant Exposure: F-Stop vs Shutter Speed

[![pPpxQ4U.jpg](https://s1.ax1x.com/2023/07/31/pPpxQ4U.jpg)](https://imgse.com/i/pPpxQ4U)

## Thin Lens Approximation

[![pPpxnH0.jpg](https://s1.ax1x.com/2023/07/31/pPpxnH0.jpg)](https://imgse.com/i/pPpxnH0)

* 薄透镜近似

* 使用透镜组来成像

* 实际的透镜很复杂，不一定理想

* 我们考虑的是理想化的透镜

[![pPp0IeS.jpg](https://s1.ax1x.com/2023/07/30/pPp0IeS.jpg)](https://imgse.com/i/pPp0IeS)

* 基本假设：
  
  * 平行光穿过透镜交于焦点
  
  * 过焦点的光穿过透镜会平行
  
  * 焦距可以修改(通过透镜组)
  
  * 穿过透镜中心的光线，方向不会改变
  
  * 焦距的倒数 = 相距的倒数 + 物距的倒数，也就是对于一个固定焦距的透镜来说，改变物距相距也一定会跟着改

[![pPp04L8.jpg](https://s1.ax1x.com/2023/07/30/pPp04L8.jpg)](https://imgse.com/i/pPp04L8)

* 通过相似三角形来获得上述的关系

* 这个叫高斯定理，也叫薄透镜的定理

[![pPp0hsf.jpg](https://s1.ax1x.com/2023/07/30/pPp0hsf.jpg)](https://imgse.com/i/pPp0hsf)

## Defocus Blur

* 和景深有关系了

### Computing Circle of Confusion(CoC)

[![pPp0fQP.jpg](https://s1.ax1x.com/2023/07/30/pPp0fQP.jpg)](https://imgse.com/i/pPp0fQP)

* 物体的一个点根据薄透镜的定理，会成像到一个点，但是感光sensor Plane比那个成像点远的话，就会打成一片，也就成了一个圆
  
  * 如果其他都确定了，那么这个C和A(aperture)有关
  
  * CoC和光圈大小成正比
  
  * 所以大光圈就会有更模糊的效果

* F数=焦距除以光圈的直径

[![pPp02RI.jpg](https://s1.ax1x.com/2023/07/30/pPp02RI.jpg)](https://imgse.com/i/pPp02RI)

* N = f/A

[![pPp0sde.jpg](https://s1.ax1x.com/2023/07/30/pPp0sde.jpg)](https://imgse.com/i/pPp0sde)

## Ray Tracing Ideal Thin Lenses

* 就是在ray tracing中模拟透镜的模糊效果

### Ray Tracing for Defocus Blur(Thin Lens)

* 首先定义感光器大小、透镜焦距、透镜大小

[![pPp0yIH.jpg](https://s1.ax1x.com/2023/07/30/pPp0yIH.jpg)](https://imgse.com/i/pPp0yIH)

* 选择将透镜放在一个距离重拍摄平面Z0远的地方，也就是物距Z0
  
  * 自然可以计算出像距Zi，也就是sensor距离透镜的距离
  
  * 也就是下面$x\prime$点来自于$x\prime\prime -> x\prime\prime\prime$打的光线，交于着色点

[![pPp0Rzt.jpg](https://s1.ax1x.com/2023/07/30/pPp0Rzt.jpg)](https://imgse.com/i/pPp0Rzt)

## Depth of Field

* 模糊是有一个范围

### Circle of Confusion for  Depth of Field

[![pPp0gJA.jpg](https://s1.ax1x.com/2023/07/30/pPp0gJA.jpg)](https://imgse.com/i/pPp0gJA)

* 景深就是指在实际的场景中有一段深度，这段深度在经过透镜后，会在成像平面附近形成一个区域，在这块区域，我们认为，产生的CoC都是足够小的

* 可以通过公式将那个DoF解出来

[![pPp0cid.jpg](https://s1.ax1x.com/2023/07/30/pPp0cid.jpg)](https://imgse.com/i/pPp0cid)
