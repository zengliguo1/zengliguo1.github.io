---
title: GAMES101-Lecture 11 Geometry 2(Curves and Surface)
date: 2023-06-30 17:00:00 +0800
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

## Explicit Representations

* Point Cloud点云
  
  * 好表达
  
  * 大数据集很有用
  
  * 经常用于转换成多边形mesh
  
  * 难以在采样不足的区域进行绘制

* Polygon Mesh
  
  * 储存顶点和多边形
  
  * 方便处理、模拟、适合采样
  
  * 数据结构更复杂
  
  * 几乎最常见的显式表示

* wavefront object
  
  * v是顶点坐标
  
  * vt是纹理坐标
  
  * vn是法线
  
  * f是点的连接关系

[![pC0yzAH.jpg](https://s1.ax1x.com/2023/06/30/pC0yzAH.jpg)](https://imgse.com/i/pC0yzAH)

## Curves

### Bezier curves

[![pC0cYsf.jpg](https://s1.ax1x.com/2023/06/30/pC0cYsf.jpg)](https://imgse.com/i/pC0cYsf)

* 定义
  
  * 起始点在p0，终点在p1
  
  * 且起始点处的切线方向为p0p1，终点处的切线方向为p2p3

### De Casteljau's algorithm

[![pC0grAe.jpg](https://s1.ax1x.com/2023/06/30/pC0grAe.jpg)](https://imgse.com/i/pC0grAe)

* 这个算法需要至少三个控制点
  
  * 需要不断的取t[0,1]来采样，连接成一条曲线
  
  * 比如t = 1/3时，取b01点使得b0b01占b0b1的1/3，取b11点使得b1b11占b1b2的1/3，然后在b01b11取点b02使得b01b02占b01b11的1/3
  
  * 那么在t = 1/3时，曲线过点b02
  
  * 以此类推

[![pC0TdG8.jpg](https://s1.ax1x.com/2023/06/30/pC0TdG8.jpg)](https://imgse.com/i/pC0TdG8)

* 三次贝塞尔曲线，需要四个点，操作跟上面的类似

[![pC0bP8f.jpg](https://s1.ax1x.com/2023/06/30/pC0bP8f.jpg)](https://imgse.com/i/pC0bP8f)

* 公式就是不断地插值，递归

[![pC0bi28.jpg](https://s1.ax1x.com/2023/06/30/pC0bi28.jpg)](https://imgse.com/i/pC0bi28)

* n是控制点的个数，B是伯恩斯坦多项式

* 在三维空间中依然适用

[![pC0b3MF.jpg](https://s1.ax1x.com/2023/06/30/pC0b3MF.jpg)](https://imgse.com/i/pC0b3MF)

* 贝塞尔曲线的性质：
  
  * 起始点和终点确定
  
  * 起始点和终点切线方向确定
  
  * 仿射变换的结果一样（对控制点先变换再画线和先画线再对线上的控制点变换），但对投影不一样
  
  * 凸包性质：曲线一定在控制点形成的凸包内

[![pC0bOJ0.jpg](https://s1.ax1x.com/2023/06/30/pC0bOJ0.jpg)](https://imgse.com/i/pC0bOJ0)

### Piecewise Bezier Curves 多段贝塞尔

[![pC0qDf0.jpg](https://s1.ax1x.com/2023/06/30/pC0qDf0.jpg)](https://imgse.com/i/pC0qDf0)

* 最常用的是三次贝塞尔曲线（4个控制点）

[![pC0qL0H.jpg](https://s1.ax1x.com/2023/06/30/pC0qL0H.jpg)](https://imgse.com/i/pC0qL0H)

* 当一个终点控制点（连接两条曲线）相邻两个控制点在一条直线上且距离该终点距离相同，那么这两条曲线实现了平滑的连接

* 如果在一条直线上但距离该终点距离不相同，那么不算平滑连接。比如一个蚂蚁爬到终点时会突然加速

[![pC0OMGt.jpg](https://s1.ax1x.com/2023/06/30/pC0OMGt.jpg)](https://imgse.com/i/pC0OMGt)

* C0连续：第一段的终点和第二段的终点重合

* C1连续：第一段的终点和第二段的终点重合，且第一段的an-1点和第二段的b1点在一条直线上，且距离重合点距离相等（相当于第一段的终点和第二段的终点的一阶导数相等）

* C2连续：C1的基础上，二阶导数再相等

### B-splines（basis splines）

* B样条不需要分段就具有局部操控性（动一个点不会整条曲线都改变）

* 极其复杂

* 非均匀有理B样条（NURBS）更复杂

## Surfaces

### Bezier surfaces

[![pC0XjAJ.jpg](https://s1.ax1x.com/2023/06/30/pC0XjAJ.jpg)](https://imgse.com/i/pC0XjAJ)

* 首先，现在有16个点，每行4个，先根据每行的4个点画出四条曲线，然后根据t，在四条线上一共会找到四个点，再根据这四个点来绘制列的曲线

[![pC0jwuT.jpg](https://s1.ax1x.com/2023/06/30/pC0jwuT.jpg)](https://imgse.com/i/pC0jwuT)

* uv上的任何一个点都可以映射到曲面上，uv可以理解成t1和t2

* 所以贝塞尔曲线是显式表示，因为它可以被映射而来

### Subdivision,simplification,regularization

[![pC0vGqO.jpg](https://s1.ax1x.com/2023/06/30/pC0vGqO.jpg)](https://imgse.com/i/pC0vGqO)
![](https://www.notion.so/4a0df59c756b440a87f9057b1c1c1729?pvs=4#4e5dbb2ae4c7464b950193f9592ac8e0)
