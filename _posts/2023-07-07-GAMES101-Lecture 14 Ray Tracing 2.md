---
title: GAMES101-Lecture 14 Ray Tracing 2
date: 2023-07-07 14:32:00 +0800
categories: [计算机图形学, GAMES101]
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

## Using AABBs to accelerate ray tracing

### Uniform grids

* 主旨：多做光线与盒子求交

* 先把场景的BB框住，然后在BB里创建格子

* 记录每个有物体（表面）存在的格子(下图中右上角少画了一个格子)

[![pCcSay4.jpg](https://s1.ax1x.com/2023/07/07/pCcSay4.jpg)](https://imgse.com/i/pCcSay4)

* 判断跟光线相交的格子里有没有物体

* 如果有物体，那么说明光线可能和该物体相交，就计算交点

* 在光线的发射过程中（对于图中的光线），只需判断当前格子的上方和右方是否为光线下一个相交的格子

[![pCcpPpT.jpg](https://s1.ax1x.com/2023/07/07/pCcpPpT.jpg)](https://imgse.com/i/pCcpPpT)

### Grid Resolution

* 不能太稀疏（比如1\*1\*1的格子，相当于没做BBAA），也不能太密集，不然和格子也求交太多次，所以要有一个均衡

[![pCcplcD.jpg](https://s1.ax1x.com/2023/07/07/pCcplcD.jpg)](https://imgse.com/i/pCcplcD)

* BBAA适合比较均匀的场景，不适合有大片空区域的场景，会产生“Teapot in a stadium”问题

### Spatial partitions(空间划分)

* 基于格子的不足之处进行优化，在物体分布稀疏的地方格子大一点，，分布密集的地方格子小一点

* 八叉树（Oct-Tree），对空间切三刀，然后如果某个小空间中物体比较多，继续切，直到空间内没物体或者物体数量差不多了，但维度再高的话，就是$2^n$叉树，这显然不好（我认为老师意思是说这么切下去还是切太多了），为解决这个问题，KD-Tree应运而生

* KD-Tree：每次砍一刀，然后再根据情况在分出的一个空间内再砍一刀，这样就类似二叉树，每次水平一刀，竖直一刀，这样划分的相对均匀一点

* BSP-Tree：不是横平竖直地砍，所以比较复杂

[![pCcF9Qf.jpg](https://s1.ax1x.com/2023/07/07/pCcF9Qf.jpg)](https://imgse.com/i/pCcF9Qf)

### KD-Tree Pre-Processing

* 在做光追之前提前划分好，在中间节点记录它之后划分了什么样的格子，在叶子节点来实际存储和格子相交的物体（三角形）

[![pCcFdOO.jpg](https://s1.ax1x.com/2023/07/07/pCcFdOO.jpg)](https://imgse.com/i/pCcFdOO)

* 首先，光线和整个BBAA*（A)有交点，那么对于子节点（也就是1和B）都判断是否有交点

* 都相交，但1也就到此为止，去计算1中所有物体是否与光线相交，然后去计算B的子节点2和C

* 还是都相交，但2也到此为止，去计算2中所有物体是否与光线相交，然后去计算C的子节点3和D

* 还是都相交，继续算

* 到4和5，其中5是没有交点的，所以就不计算了

* 有几个问题：
  
  * 如何判断包围盒的格子到底和哪些三角形相交，这个问题比较困难处理
  
  * 一个物体与多个包围盒相交，也就是多个叶子节点都要储存该物体

### Object Partitions & Bounding Volume Hierarchy(BVH)

* 解决了KD-Tree的两个问题，既不用求交，而且一个三角形不会与多个包围盒相交，只会存在一个包围盒中

* 但问题是包围盒有可能相交，怎么划分很讲究

[![pCc3MX4.jpg](https://s1.ax1x.com/2023/07/07/pCc3MX4.jpg)](https://imgse.com/i/pCc3MX4)

* 怎样区分一个节点：
  
  * 选择一个维度去划分（x、y、z轴）
  
  * 总是沿着最长的轴去切
  
  * 区分节点（切）的时候，从中间物体的位置切，快速找出中位数，使用快速选择算法，只需要O(n)的复杂度

* 如果是动态场景，就需要重新计算BVH了

* BVH伪代码：

[![pCc8pU1.jpg](https://s1.ax1x.com/2023/07/07/pCc8pU1.jpg)](https://imgse.com/i/pCc8pU1)

### Spatial VS Object Partitions

* 空间划分和物体划分的区别

* 空间：
  
  * 划分了不重叠的空间区域
  
  * 一个物体可能会被包含在多个空间中

* 物体：
  
  * 物体被划分到不同的子集中
  
  * 包围盒可能会重叠

## Basic radiometry (辐射度量学)

### Motivation

* 为光赋予物理意义

* 定义了一些列的方法和单位来描述光照

* 准确计量光在空间中的各种属性
  
  * Radiant flux：光通量，单位时间内通过的光总量，单位watt或者lumen（lm）
  
  * intensity：光强，单位立方角光通量，单位Watt/steradians，也就是candelas
  
  * irradiance：辉度，单位面积光通量，瓦特/平方米
  
  * radiance：光亮度

### Radiant Energy and Flux(Power)

[![pCcJ91K.jpg](https://s1.ax1x.com/2023/07/07/pCcJ91K.jpg)](https://imgse.com/i/pCcJ91K)

* Radiant Energy：可以理解为光的能量，单位是焦耳

* Radiant Flux(power)：单位时间的能量，单位是瓦特

* Flux（另外一种解释）：光打到一个感光的平面，单位时间内通过的光子的数量

[![pCcJC6O.jpg](https://s1.ax1x.com/2023/07/07/pCcJC6O.jpg)](https://imgse.com/i/pCcJC6O)

### Important Light Measurements of Interest

[![pCcJRgK.jpg](https://s1.ax1x.com/2023/07/07/pCcJRgK.jpg)](https://imgse.com/i/pCcJRgK)

* Radiant Intensity：光源散发的光

* irradiance：一个平面接收的光

* radiance：在传播中的光

### Radiant Intensity

* 单位立体角的能量

[![pCcJ4De.jpg](https://s1.ax1x.com/2023/07/07/pCcJ4De.jpg)](https://imgse.com/i/pCcJ4De)

* 二维中角度一般指弧度，单位是radians

* 三维中立体角：球上一块面积除以半径的平方，球有$4\pi$的立体角，单位是steradians

[![pCcYZb4.jpg](https://s1.ax1x.com/2023/07/07/pCcYZb4.jpg)](https://imgse.com/i/pCcYZb4)

* 单位立体角/微分立体角：微分面积/半径平方

[![pCcYfGq.jpg](https://s1.ax1x.com/2023/07/07/pCcYfGq.jpg)](https://imgse.com/i/pCcYfGq)

* 之后会用单位立体角w来表示一个单位向量

[![pCcY4zV.jpg](https://s1.ax1x.com/2023/07/07/pCcY4zV.jpg)](https://imgse.com/i/pCcY4zV)

* Radiant Intensity就是单位立体角的能量

[![pCcYIMT.jpg](https://s1.ax1x.com/2023/07/07/pCcYIMT.jpg)](https://imgse.com/i/pCcYIMT)
