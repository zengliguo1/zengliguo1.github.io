---
title: GAMES101-Lecture 03 Transformation
date: 2022-07-23 22:20:00 +0800
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

### 1. Scale

$$
\left[
\begin{matrix}
x\prime\\
y\prime
\end{matrix}\right]=
\left[
\begin{matrix}
s_x & 0\\
0 & s_y
\end{matrix}\right]
\left[
\begin{matrix}
x\\
y
\end{matrix}\right]
$$

### 2.Reflection Matrix

- 沿y轴翻转

$$
\left[
\begin{matrix}
x\prime\\
y\prime
\end{matrix}\right]=
\left[
\begin{matrix}
-1 & 0\\
0 & 1
\end{matrix}\right]
\left[
\begin{matrix}
x\\
y
\end{matrix}\right]
$$

### 3. Shear Matrix

- 水平剪切，水平方向上都移动了a×y

$$
\left[
\begin{matrix}
x\prime\\
y\prime
\end{matrix}\right]=
\left[
\begin{matrix}
1 & a\\
0 & 1
\end{matrix}\right]
\left[
\begin{matrix}
x\\
y
\end{matrix}\right]
$$

### 4. Rotate(about (0,0)，CCW by default)

* 绕原点旋转
- 默认逆时针旋转，可由特殊点推导出

$$
R_\theta=
\left[
\begin{matrix}
\cos\theta & -\sin\theta\\
\sin\theta & \cos\theta
\end{matrix}\right]
$$

* 用一个正方形的两个点来算就能很快算出这个旋转矩阵（1，0）和（0，1）

### 5. Linear Transforms = Matrices(same dimension)

* 变换都可以写成线性变换

$$
\left[
\begin{matrix}
x\prime\\
y\prime
\end{matrix}\right]=
\left[
\begin{matrix}
a & b\\
c & d
\end{matrix}\right]
\left[
\begin{matrix}
x\\
y
\end{matrix}\right]

$$

$$
x\prime=Mx
$$

### 6. Transiation

- 平移变换不是线性变换，但我们不希望把平移变换当成一个特殊的变换

- 引入齐次坐标Homogenous Coordinates
  
  $$
  \left(
\begin{matrix}
x\prime\\
y\prime\\
w\prime
\end{matrix}
\right)=
\left(
\begin{matrix}
1 & 0 & t_x\\
0 & 1 & t_y\\
0 & 0 & 1
\end{matrix}
\right)\cdot
\left(
\begin{matrix}
x\\
y\\
1
\end{matrix}
\right)=
\left(
\begin{matrix}
x + t_x\\
y + t_y\\
1
\end{matrix}
\right)
  $$

- 添加第三位坐标(向量是0，因为向量具有平移不变性)
  
  - 2D point = $(x,y,1)^T$
  
  - 2D vector = $(x,y,0)^T$

- 如果w坐标是1或者0，点和向量的运算也是有效的
  
  - vector + vector = vector（z还是0）
  
  - point - point = vector（z = 1 - 1 = 0）
  
  - point + vector = point（一个点沿着一个向量走，最后还是一个点）
  
  - point + point = 这两个点的中点(因为第三位数1+1=2，最后的结果想转换成第三位是1需要x和y同时除以2)
    
    $$
    \left(
\begin{matrix}
x\\
y\\
w
\end{matrix}
\right)
就是2D点
\left(
\begin{matrix}
\frac{x}{w}\\
\frac{y}{w}\\
1
\end{matrix}
\right),w\neq0
    $$

### 7.Affine Transformations仿射变换

- Affine map = linear map + translation（线性变化+平移）

- 仿射变换都可以转换为齐次坐标的形式

- 在表示二维情况下的仿射变换时，齐次坐标对应的最后一行才是(0,0,1)，其他情况下有其他的意义

### 8.Inverse Transform

$$
M^-1
$$

### 9. Composite Transform(复合变换)

* 矩阵没有交换律但有结合律
- 从右到左逐个应用矩阵

### 10. Decomposing Complex Transforms

- 绕C点旋转：先平移，使C移到原点，旋转后，再移回去（C点可以是边缘的也可以是中心点）
  
  $T(c)\cdot R(\alpha)\cdot T(-c)$

### 11. 3D Transforms

- 再一次使用齐次坐标

- $3D point = (x,y,z,1)^T$

- $3D vector = (x,y,z,0)^T$

- 使用4×4的矩阵表示仿射变换

$$
\left(
\begin{matrix}
x\prime\\
y\prime\\
z\prime\\
1
\end{matrix}
\right)=
\left(
\begin{matrix}
a & b & c & t_x\\
d & e & f & t_y\\
g & h & i & t_z\\
0 & 0 & 0 & 1
\end{matrix}
\right)\cdot
\left(
\begin{matrix}
x\\
y\\
z\\
1
\end{matrix}
\right)
$$

- 是先做的旋转再做的平移

### 12. 补充

$$
R_\theta=
\left(
\begin{matrix}
\cos\theta & -\sin\theta\\
\sin\theta & \cos\theta
\end{matrix}\right)\\

$$

$$
R_{-\theta}=
\left(
\begin{matrix}
\cos\theta & \sin\theta\\
-\sin\theta & \cos\theta
\end{matrix}\right)=R_\theta^T\\

$$

$$
R_\theta^T=R_\theta^{-1}
$$

* 旋转矩阵从定义的角度来看，旋转矩阵的逆矩阵（也就是旋转-$\theta$角）与旋转矩阵的转置是相等的
