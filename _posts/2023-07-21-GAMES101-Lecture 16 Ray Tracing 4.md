---
title: GAMES101-Lecture 16 Ray Tracing 4(Monte Carlo Path Tracing)
date: 2023-07-21 15:44:00 +0800
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

## Monte Carlo Integration

* WHY:我们想要算一个函数的积分，但积分难以写出表达公式，所以就用蒙特卡洛的方法求出定积分那个数值来

* WHAT&HOW:就是相当于离散化后求面积

[![pCHj5fU.jpg](https://s1.ax1x.com/2023/07/21/pCHj5fU.jpg)](https://imgse.com/i/pCHj5fU)

* 其中f(Xi)相当于在Xi处的函数值，除以该函数值出现的概率，就相当于求出了以f(Xi)的值为长，概率密度函数p(Xi)的值为高，的一个矩形面积，就像这样不断采样，在积分域中采样n个，最后除以n，就是蒙特卡洛积分，任何积分都可以这么做

[![pCHjvtK.jpg](https://s1.ax1x.com/2023/07/21/pCHjvtK.jpg)](https://imgse.com/i/pCHjvtK)

[![pCHvKXj.jpg](https://s1.ax1x.com/2023/07/21/pCHvKXj.jpg)](https://imgse.com/i/pCHvKXj)

## Path Tracing

* 提出path tracing就是为了解决whitted-style ray tracing一些不正确的地方

* whitted-style中ray打到diffuse就会停，但是实际上不应该停住

[![pCHvc9O.jpg](https://s1.ax1x.com/2023/07/21/pCHvc9O.jpg)](https://imgse.com/i/pCHvc9O)

* render equation是正确的
  
  * 需要计算半球内的积分
  
  * 需要递归

### A Simple Monte Carlo Solution

* 把自发光去掉

[![pCHzFFP.jpg](https://s1.ax1x.com/2023/07/21/pCHzFFP.jpg)](https://imgse.com/i/pCHzFFP)

* 其中pdf为啥是$1/2\pi$，这是因为，球的立体角是$4\pi$，半球就是一半

* 使用蒙特卡洛：

[![pCHzjkq.jpg](https://s1.ax1x.com/2023/07/21/pCHzjkq.jpg)](https://imgse.com/i/pCHzjkq)

* 直接光照（没有考虑间接光照）的伪代码：

[![pCbSlBd.jpg](https://s1.ax1x.com/2023/07/21/pCbSlBd.jpg)](https://imgse.com/i/pCbSlBd)

### Global Illumination

* 间接光照，使用递归即可

* 伪代码如下：

[![pCbkzy6.jpg](https://s1.ax1x.com/2023/07/21/pCbkzy6.jpg)](https://imgse.com/i/pCbkzy6)

* 但这么做，会产生两个问题

[![pCbmcwV.jpg](https://s1.ax1x.com/2023/07/21/pCbmcwV.jpg)](https://imgse.com/i/pCbmcwV)

* 首先是光线的数量会爆炸，数量级会不断增加，当n（也就是随机取的光线数量）为1时，不会爆炸，此时就叫做path tracing，问题就是噪声非常大
  
  * 所以就需要向每个像素投射更多路径然后求平均，投射时也采用蒙特卡洛积分，向像素点随机选择n个位置

[![pCbm6e0.jpg](https://s1.ax1x.com/2023/07/21/pCbm6e0.jpg)](https://imgse.com/i/pCbm6e0)

* 递归需要一个终止条件
  
  * 此时，聪明的人类提出了一种方法：Russian Roulette俄罗斯轮盘赌

### Russian Roulette (RR)

* 先前，我们总是会发射一条光线到shading point，然后道德shading结果Lo

* 现在，我们假设人为地设置一个概率P(0<P<1)，在概率P的可能下会发射一条光线并得到shading结果为Lo/P，在概率1-P的可能下不发射光线，结果也就是0

* 通过这种方法，仍然可以期望得到Lo

$E=p*(Lo/P)+(1-P)*0=Lo$

* 伪代码如下：

[![pCbmwWQ.jpg](https://s1.ax1x.com/2023/07/21/pCbmwWQ.jpg)](https://imgse.com/i/pCbmwWQ)

* 但这并不是很高效，因为有的光源很小，所以采样的时候发射很多光线才有可能打到，但这就浪费了很多其他光线，所以就引出了下面的方法：在光源上采样

[![pCbm0zj.jpg](https://s1.ax1x.com/2023/07/21/pCbm0zj.jpg)](https://imgse.com/i/pCbm0zj)

[![pCbmDQs.jpg](https://s1.ax1x.com/2023/07/21/pCbmDQs.jpg)](https://imgse.com/i/pCbmDQs)

### Sampling the Light(pure math)

[![pCbmryn.jpg](https://s1.ax1x.com/2023/07/21/pCbmryn.jpg)](https://imgse.com/i/pCbmryn)

* 需要把渲染方程写成在光源上的积分

* 也就是说需要找到dw和dA的关系
  
  * 首先需要将光源转向着色点
  
  * 然后除以dA和dw距离的平方

* 然后就能很快解决蒙特卡洛积分

* 所以现在的path tracing就分为两部分
  
  * 一部分光线就进行对光源的采样积分
  
  * 另外一部分反射不到光源的，就进行俄罗斯轮盘赌的蒙特卡洛积分

* 最后还有一步，要判断光源与着色点之间是不是有其他物体阻挡

* 伪代码：

[![pCbmsLq.jpg](https://s1.ax1x.com/2023/07/21/pCbmsLq.jpg)](https://imgse.com/i/pCbmsLq)

### 尚存的问题

* 如何在半球进行均匀的采样，如何对一个函数采样

* 目前用的均匀的pdf是最好的方法吗

* 随机数有好坏之分吗（low discrepancy sequences的随机数生成的比较均匀）

* 结合对半球和光源的采样（multiple imp. sampling）

* 像素的radiance是所有路径radiance的平均，为什么？(pixel reconstruction filter)

* 像素的radiance就是像素的颜色吗？no（gamma correction, curves, color space）
