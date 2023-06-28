---
title: GAMES101-Lecture 10 Geometry 1
date: 2023-06-27 17:00:00 +0800
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

## 1. Applications of Textures

* In modern GPUs, texture = memory + range query (filtering)

* Environment Map环境贴图

* Spherical Map、Cube Map

[![pCd3faQ.jpg](https://s1.ax1x.com/2023/06/28/pCd3faQ.jpg)](https://imgse.com/i/pCd3faQ)

* **Bump Mapping**
  
  * 定义的是纹理上任意一个点，高度的相对移动
  
  * [![pCd3xi9.jpg](https://s1.ax1x.com/2023/06/28/pCd3xi9.jpg)](https://imgse.com/i/pCd3xi9)
  
  * 通过凹凸贴图（也可以说是高度贴图），改变了原来的点的高度（实际没改，只是用来算法线），从而改变了法线的方向，制造了凹凸的效果
  
  * [![pCdGNAe.jpg](https://s1.ax1x.com/2023/06/28/pCdGNAe.jpg)](https://imgse.com/i/pCdGNAe)
  
  * 对平面来说：该点的切线方向是(1,dp)，dp就是凹凸贴图存的值，那么该点的法线方向就是转90°，也就是(-dp,1)
  
  * [![pCd0bEq.jpg](https://s1.ax1x.com/2023/06/28/pCd0bEq.jpg)](https://imgse.com/i/pCd0bEq)
  
  * 在三维空间中，假设一个法向量(0,0,1)，根据凹凸图某一点uv的梯度值来计算出扰乱后的切线，最后旋转就成了扰乱后的法线也就是(-dp/du,-dp/dv,1)，因为假设的法向量(0,0,1)在local坐标系（局部坐标系stn），所以之后还需要转换

* **Displacement mapping**

* [![pCdBwaq.jpg](https://s1.ax1x.com/2023/06/28/pCdBwaq.jpg)](https://imgse.com/i/pCdBwaq)

* 3d生成噪声作为贴图

* [![pCdB0I0.jpg](https://s1.ax1x.com/2023/06/28/pCdB0I0.jpg)](https://imgse.com/i/pCdB0I0)

* 环境光遮罩贴图

* [![pCdBrGT.jpg](https://s1.ax1x.com/2023/06/28/pCdBrGT.jpg)](https://imgse.com/i/pCdBrGT)

* 体渲染

* [![pCdBsRU.jpg](https://s1.ax1x.com/2023/06/28/pCdBsRU.jpg)](https://imgse.com/i/pCdBsRU)

## 2. Geometry

[![pCdDeYV.jpg](https://s1.ax1x.com/2023/06/28/pCdDeYV.jpg)](https://imgse.com/i/pCdDeYV)

* implicit隐式几何
  
  * 点满足一些特殊的关系，比如一个球的方程，并不给实际的点
  
  * [![pCdD2p8.jpg](https://s1.ax1x.com/2023/06/28/pCdD2p8.jpg)](https://imgse.com/i/pCdD2p8)
  
  * [![pCdDcff.jpg](https://s1.ax1x.com/2023/06/28/pCdDcff.jpg)](https://imgse.com/i/pCdDcff)
  
  * 难点在于哪些点符合这个关系，很难从式子看出来是什么模型。但是，给定一个点，判断这个点在不在这个面上，很容易。直接把点代入式子，如果是0在面上，大于0在外，小于0在内

* explicit显式几何
  
  [![pCdDTkq.jpg](https://s1.ax1x.com/2023/06/28/pCdDTkq.jpg)](https://imgse.com/i/pCdDTkq)

* 要么直接给出，要么通过参数映射的方式给出

* 把所有uv找一遍就知道在三维空间中长什么样，所以采样更简单；但是判断某个点是否在平面的里外，很难

* [![pCdDI7n.jpg](https://s1.ax1x.com/2023/06/28/pCdDI7n.jpg)](https://imgse.com/i/pCdDI7n)

* [![pCdD50s.jpg](https://s1.ax1x.com/2023/06/28/pCdD50s.jpg)](https://imgse.com/i/pCdD50s)

* [![pCdrnAI.jpg](https://s1.ax1x.com/2023/06/28/pCdrnAI.jpg)](https://imgse.com/i/pCdrnAI)

* **CSG(constructive solid geometry)(implicit)**
  
  * 通过基本的几何的组合来构造几何
  
  * [![pCdrNEn.jpg](https://s1.ax1x.com/2023/06/28/pCdrNEn.jpg)](https://imgse.com/i/pCdrNEn)

* Distance Functions(implicit)
  [![pCdsoWV.jpg](https://s1.ax1x.com/2023/06/28/pCdsoWV.jpg)](https://imgse.com/i/pCdsoWV)
  
  * 距离函数是指空间中任意一个点到一个模型的最短距离，如果这个点在模型的面外就是正的，如果在面内，就是负的
  
  * [![pCdsIJ0.jpg](https://s1.ax1x.com/2023/06/28/pCdsIJ0.jpg)](https://imgse.com/i/pCdsIJ0)
  
  * A是一个物体挡住窗口1/3时候的一张图，B是该物体挡住窗口的2/3时候的一张图，现在的目标是求出这两个状态中间状态的样子
    
    * 如果拿两张图做一个线性blend(相加除以二)，那么左边黑，中间灰，右边白
    
    * 但是我们希望的结果是运动的中间态也就是左边一半是黑的，右边一半是白的
  
  * SDF是signed distance function。对A和B计算距离函数也就是变换成SDF(A)和SDF(B)，再通过距离函数来blend，这样的话结果中间就会是0，最后再恢复
  
  * 
  
  * [![pCds5iq.jpg](https://s1.ax1x.com/2023/06/28/pCds5iq.jpg)](https://imgse.com/i/pCds5iq)

* Level Set Methods水平集方法（类似距离函数）

* [![pCdg9S0.jpg](https://s1.ax1x.com/2023/06/28/pCdg9S0.jpg)](https://imgse.com/i/pCdg9S0)

* Fractals分形(自相似)(递归)


