---
title: GAMES101-Lecture 15 Ray Tracing 3(Light Transport & Global Illumination)
date: 2023-07-10 15:07:00 +0800
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

## Radiometry（后半部分）

### Irradiance

* 单位面积上的能量，其中光的能量方向必须得与表面点垂直
* 单位面积指的是接受辐射的微小面积dA

[![pC26nmj.jpg](https://s1.ax1x.com/2023/07/10/pC26nmj.jpg)](https://imgse.com/i/pC26nmj)

* 对于之前的修正：

[![pC2hQr8.jpg](https://s1.ax1x.com/2023/07/10/pC2hQr8.jpg)](https://imgse.com/i/pC2hQr8)

### Radiance

* 单位立体角、单位面积（projected unit area）上的光的能量，相当于单位面积上的Radiant Intensity，也相当于单位solid angle的Irradiance
* 单位面积指的是接受辐射的微小面积dA
* 往solid angle方向辐射能量，$\theta$指的是面的法线和solid angle的方向夹角

[![pC2WPRx.jpg](https://s1.ax1x.com/2023/07/10/pC2WPRx.jpg)](https://imgse.com/i/pC2WPRx)

### Incident Radiance（入射）

* 单位立体角的Irradiance到达某个单位面积平面的能量

* 这里单位面积指的是光打到的平面，也就是接受辐射的微小面积dA

[![pC2W56K.jpg](https://s1.ax1x.com/2023/07/10/pC2W56K.jpg)](https://imgse.com/i/pC2W56K)

### Exiting Radiance

* 单位面积上的Radiant Intensity

* 这个跟上面的Radiance一样，单位面积也是接受辐射的微小面积dA，其实就是Radiance的另一种说法

[![pC2fpnS.jpg](https://s1.ax1x.com/2023/07/10/pC2fpnS.jpg)](https://imgse.com/i/pC2fpnS)

### Irradiance vs. Radiance

[![pC2hWM6.jpg](https://s1.ax1x.com/2023/07/10/pC2hWM6.jpg)](https://imgse.com/i/pC2hWM6)

* 两者之间的差距就是方向性，irradiance接受的辐射是来自四面八方的，最后积分积一下，而Radiance接受的是来自单位角方向的辐射

## Light transport

### Bidirectional Reflectance Distribution Function(BRDF双向反射分布函数)

* 为了解决，已知入射光能量和角度，射到物体表面会向各个方向辐射，辐射出去的能量和角度是不一样的，这个方法能求出给定方向的辐射的能量是多少，比如辐射到相机那里会有多少能量

* 也可以这么理解，一个表面把光线的能量吸收了，然后要再发出去

[![pC2qbWD.jpg](https://s1.ax1x.com/2023/07/10/pC2qbWD.jpg)](https://imgse.com/i/pC2qbWD)

[![pC2qLSe.jpg](https://s1.ax1x.com/2023/07/10/pC2qLSe.jpg)](https://imgse.com/i/pC2qLSe)

### The reflection equation

* 下面这个公式是计算来自四面八方的入射光在Lr方向的能量贡献

[![pC2LPfS.jpg](https://s1.ax1x.com/2023/07/10/pC2LPfS.jpg)](https://imgse.com/i/pC2LPfS)

#### Chalenge：Recursive Equation

* 因为光线可能会弹射多次，因而一个着色点受到的光不一定只是来自光源，可能也来自其他物体，所以在计算的时候会是递归式的

### The rendering equation

* 渲染公式就是自发光+反射光

* $H^2$和$\Omega+$都是表示半球

* 虽然入射方向wi指向球心，但把w的方向都算成从球心（着色点）指向球外

* 把$\cos\theta$写成了$n\cdot w_i$法线乘以radiance方向

[![pC2LzjJ.jpg](https://s1.ax1x.com/2023/07/10/pC2LzjJ.jpg)](https://imgse.com/i/pC2LzjJ)

* 一个点光源的简单情况：

[![pC2v6xS.jpg](https://s1.ax1x.com/2023/07/10/pC2v6xS.jpg)](https://imgse.com/i/pC2v6xS)

* 多个点光源

[![pC2vy28.jpg](https://s1.ax1x.com/2023/07/10/pC2vy28.jpg)](https://imgse.com/i/pC2vy28)

* 面光源（积分一下）

[![pC2vs8f.jpg](https://s1.ax1x.com/2023/07/10/pC2vs8f.jpg)](https://imgse.com/i/pC2vs8f)

* 其他物体反射

[![pC2vabd.jpg](https://s1.ax1x.com/2023/07/10/pC2vabd.jpg)](https://imgse.com/i/pC2vabd)

* 简写渲染方程，L是要算的东西，E是发光，K是反射操作符

[![pC2vrPP.jpg](https://s1.ax1x.com/2023/07/10/pC2vrPP.jpg)](https://imgse.com/i/pC2vrPP)

* 然后推导，其中右边的L是(E+KL)，K算子具有泰勒展开的性质

[![pC2vB5t.jpg](https://s1.ax1x.com/2023/07/10/pC2vB5t.jpg)](https://imgse.com/i/pC2vB5t)

[![pC2v0UI.jpg](https://s1.ax1x.com/2023/07/10/pC2v0UI.jpg)](https://imgse.com/i/pC2v0UI)

## Global illumination

* 全局光照是直接和间接光照的集合

* 全局光照最终会收敛到一个亮度
