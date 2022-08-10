---
title: GAMES101-Lecture 07 Shading 1(Illumination, Shading and Graphics Pipeline)
date: 2022-08-09 18:08:00 +0800
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

### 1. Painter's Algorithm

- 从远到近画，把深度排序（O(nlogn)），然后画出来

### 2. Z-Buffer

- 每个像素永远去记录这个像素所表示的几何最浅的深度

- 生成最终渲染图的同时，还需要另一个图像，储存像素所看到的最浅的深度信息（深度图/深度缓存）

- Z-Buffer Algorithm
  
  ```cpp
  //初始化深度缓冲为正无穷
  //在光栅化过程中：
  for(each triangle T)
  {
      for(each sample(x,y,z)in T)
      {
          if(z<zbuffer[x,y])//目前最近的采样点
              {
                  framebuffer[x,y]=rgb;//更新颜色
                  zbuffer[x,y]=z;//跟新深度图中该像素的深度    
              }
      }
  }
  ```

- 复杂度：O(n)（n个三角形）

- 这是一个非常重要的算法，应用在所有的GPU上

### 3. Shading（着色）

* 对一个物体应用一个材质
* 一个简单的着色模型：Blinn-Phong Reflectance Model
* 镜面高光、漫反射、环境光照

### 4. Shading is Local（着色具有局部性，不管阴影）

    先定义一些变量，每一个shading point都有以下inputs（方向都是单位向量）

* 注视方向，**v**

* 表面法线，**n**

* 光照方向，**l**

* 表面参数，颜色、亮度

### 5. Diffuse Reflection

* 反射到四面八方

* Lambert's cosine law:$\cos\theta = \vec{l}*\vec{n}$

* shading point处接收到的光照能量与cosθ成正比

* 点光源的光线传播的能量衰减与距离半径r^2成反比

* $k_d$是漫反射系数（颜色吸收，1意味着不吸收能量，全部反射出去，0意味着全吸收，表现为黑色；如果是一个向量，就可以表示颜色了），$I/r^2$是光线到达着色点处的能量大小
  
  $$
  L_d = k_d(I/r^2)max(0, \vec{n}\cdot\vec{l})
  $$

* 当光线方向和法线方向夹角大于90°时不考虑这个光

* 无论从哪个方向看，同一个着色点的亮度是一样的
