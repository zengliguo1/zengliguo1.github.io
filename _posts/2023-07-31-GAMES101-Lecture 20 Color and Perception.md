---
title: GAMES101-Lecture 20 Color and Perception
date: 2023-07-31 15:49:00 +0800
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

## Light Field / Lumigraph

* 光场：记录所有光可能的位置和方向

[![pP98AG8.jpg](https://s1.ax1x.com/2023/07/31/pP98AG8.jpg)](https://imgse.com/i/pP98AG8)

### The Plenoptic Function(全光函数)

* 从一个地方，往某个方向看（极坐标）

* 然后添加一个变量-波长（也就是添加了颜色）

* 然后再扩展一个时间t，那么现在就是电影

* 然后再扩展一个三维空间中任意的位置，也就是全息电影

[![pP98ERS.jpg](https://s1.ax1x.com/2023/07/31/pP98ERS.jpg)](https://imgse.com/i/pP98ERS)

* 首先定义光线：5D，极坐标2个参数代表方向，空间位置3个参数

* 可以缩减到4D，也就是位置改成2个，因为光路可逆，所以看物体，也就相当于物体的包围盒上一点与视点的连线，所以包围盒的坐标（2个参数）当成空间中的坐标

[![pP98iIP.jpg](https://s1.ax1x.com/2023/07/31/pP98iIP.jpg)](https://imgse.com/i/pP98iIP)

* 光场的好处就是从任意一个位置看向物体，都可以在光场中查询到物体的颜色值

### Lumigraph - Organization

[![pP98kPf.jpg](https://s1.ax1x.com/2023/07/31/pP98kPf.jpg)](https://imgse.com/i/pP98kPf)

* 参数化表达方法，再找一个平面，将两个平面的两个点进行计算，就能找到方向

* st是世界

* 固定uv一个点，那么在st上就是看的uv上的固定的一个点，我觉得也可以理解成人在uv上看，看向st，看到的是整个世界的东西

* 固定st一个点，看向uv上每一个点，看到的是对于同一个物体，不同方向是什么，理解为人在uv上看向st，从各个方向看st的一个位置，相当于将st的irradiance拆成了从uv发射向st的radiance

[![pP98Pat.jpg](https://s1.ax1x.com/2023/07/31/pP98Pat.jpg)](https://imgse.com/i/pP98Pat)

### Integral Imaging ("Fly's Eye" Lenslets)

* 光场摄像机，将一个像素收到的irradiance转换成radiance，某个方向的光经过透镜，radiance到一个点

[![pP93vxe.jpg](https://s1.ax1x.com/2023/07/31/pP93vxe.jpg)](https://imgse.com/i/pP93vxe)

## Light Field Camera

### The Lytro Light Field Camera

* 先拍照，支持后期聚焦，后期改光圈

* 原理就是把原本吸收irradiance的像素改成微透镜，把irradiance分散到不同的方向去，在后面记录下来

[![pP98CVI.jpg](https://s1.ax1x.com/2023/07/31/pP98CVI.jpg)](https://imgse.com/i/pP98CVI)

* 有了光场之后就可以虚拟地移动相机的位置

* 缺点：
  
  * 光场摄像机有分辨率不足的问题
  
  * 高成本

## Color

### Physical Basis of Color

#### Spectrum of Light

* 不同的波长对应不同的折射率

[![pP98Srd.jpg](https://s1.ax1x.com/2023/07/31/pP98Srd.jpg)](https://imgse.com/i/pP98Srd)

#### Spectral Power Distribution(SPD)

* 光线在不同的波长，强度是多少

* 不同的光有不同的SPD

* SPD有线性的性质

[![pP93zKH.jpg](https://s1.ax1x.com/2023/07/31/pP93zKH.jpg)](https://imgse.com/i/pP93zKH)

### Biological Basis of Color

* 视网膜上有感光细胞
  
  * 棒状细胞感知光强度
  
  * 锥形细胞感知颜色
    
    * 又分成三种不同的细胞
    
    * S细胞感知小波长，高频率的光
    
    * M感知中波长
    
    * L感知长波长

[![pP98pqA.jpg](https://s1.ax1x.com/2023/07/31/pP98pqA.jpg)](https://imgse.com/i/pP98pqA)

### Tristimulus Theory of Color

* SPD和人们感知细胞的积分得到的就是感知到的颜色

### Metamerism(同色异谱)

* 两种SPD不同的光，但被感知到的颜色确实相同的

* 通过一些方法混合成人类看起来一样的光，其背后的光谱不一定相同

### Color Reproduction / Matching

* 加色系统

* 加色系数可能是负数

#### CIE RGB Color Matching Experiment

* 每种波长的颜色由三种颜色按不同系数相加得到

### Color Space

* Standardized RGB(sRGB)
  
  * 广泛应用于成像设备
  
  * 色域是有限的

* CIE XYZ
  
  * Y一定程度上表示颜色的亮度
  
  * 将XYZ归一化，得到xyz，三者加起来是1，这样的话可视化xy即可
  
  * 固定Y，可视化x，y

### Perceptually Organized Color Spaces

#### HSV Color Space

* 专为艺术家而定

* 色调、饱和度、亮度

#### CIELAB Space

* L亮度

* a轴两端是红色和绿色

* b轴两端是蓝色和黄色

* LAB认为一个轴两端是互补色

* 人脑会自动处理互补色（好玩

### CMYK: A Subtractive Color Space

* 减色系统

* 蓝绿色、品红色、黄色和黑色

* 为什么一定要黑色，因为印刷要考虑成本，黑色墨水好造，便宜
