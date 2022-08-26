---
title: GAMES101-Lecture 08 Shading2(Shading,Pipeline and Texture Mapping)
date: 2022-08-25 20:53:00 +0800
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

### 1. Specular Term(Blinn-Phong)

- 观察方向和镜面反射方向接近时，就能看到高光项

- 注视方向点乘光照方向就能得到半程向量的方向，如果h和n（法线方向）接近，就说明和镜面反射方向很接近，就能看到高光

- $$
  \pmb{h}(半程向量)=bisector(\pmb{v},\pmb{l})
\\
=\frac{\pmb{v}+\pmb{l}}
{\lVert \pmb{v}+\pmb{l}\rVert}
  $$

- Ls时镜面反射光强度，Ks时镜面系数

- $$
  L_s = k_s(I/r^2)max(0,cos\alpha)^p
\\
=k_s(I/r^2)max(0,\pmb{n\cdot h})^p
  $$

- 直接算R（镜面反射方向）和V向量的点乘是Phong反射模型，半程向量和法向量做点乘叫Blinn-Phong，这是个改进，因为半程向量相对于反射方向向量来说更好算

- 加个p的指数是为了降低余弦的“容忍度”，p越大，那么度数增加一点就会让max(0,\pmb{n\cdot h})^p值下降很多，意味着，偏很小的角度就看不到高光了，p通常取100到200。相同物体，同一方向去看的话，随着p增大，高光范围会逐渐缩小成一个点

### 2. Ambient Term

- 假设来自四面八方的环境光照强度一样，那么环境光照强度就是一个常数（实际上不是这样）

- $$
  L_a = k_aI_a
  $$

### 3. Blinn-Phong Reflection Model

- Ambient+Diffuse+Specular=Blinn-Phong Reflection

- $$
  L = L_a+L_d+L_s
\\
=k_aI_a+
k_d(I/r^2)max(0,\pmb{n\cdot l})
+k_s(I/r^2)max(0,\pmb{n\cdot h})^p
  $$

### 4.Shading Frequencies(着色频率)

- 要把着色应用在哪些点上

- **Shade each triangle(flat shading)**

- 计算每个三角形的法线

- **Shade ezch vertex(Gouraud shading)**

- 求每个顶点的法线方向，根据每个顶点的颜色，通过插值来计算顶点内部颜色的大小

- **Shade each pixel(Phong shading)(这是一种着色频率)**

- 根据三角形三个顶点的法线，插值求出三角形内部每个像素的法线方向，对每个像素进行着色

- 当面出现的频率较高时，使用平滑着色也可以。

- 当面数超过像素数，Phong着色开销自然也会比平滑着色小了

### 5. Defining Per-vertex Normal Vectors

- 1.考虑是一个球，直接圆心到顶点

- 2.顶点相邻面的法线求个平均(根据三角形面积加权)
  
  $$
  N_v = \frac{\sum_iN_i}{\lVert \sum_iN_i \rVert}
  $$

### 6. Graphics(Real-time Rendering)Pipline

- application:Input vertices in 3D space

- Vertex Processing(Vertex Stream): Vertices positioned in screen space

- Trangle Processing(Triangle Stream):Trangles positioned in screen space

- Rasterization(Fragment Stream):Fragments(one per covered sample)一个fragment就是一个像素

- Fragment Processing(Shaded Fragments):Shaded Fragments

- Frambuffer Operations

- Display:Output image(pixels)

- mvp变换在Vertex Processing阶段

- 对每个像素采样，在Rasterization阶段

- 判定可见性（z-buffer）在Fragment Processing阶段

- 如果是顶点着色，那么在Vertex Processing阶段，如果是Phong着色，在Fragment Processing阶段

### 7. Shader Programs

- shader程序应用于每一个像素（不用写for循环）

- 着色器有两种，顶点着色器和像素/片段着色器

- 像素着色器：算像素最后的颜色

### 8. Texture Mapping

- 任何三维物体的表面都是二维的

- UV图（纹理图），u和v的范围都在[0,1]

- 三角形每个顶点都对应着一个uv

- tiled textures（无缝衔接的纹理）
