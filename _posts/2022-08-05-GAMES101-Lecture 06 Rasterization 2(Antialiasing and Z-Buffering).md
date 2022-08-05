---
title: GAMES101-Lecture 06 Rasterization 2(Antialiasing and Z-Buffering)
date: 2022-08-05 20:59:00 +0800
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

### 1. Sample Artifacts

- 锯齿、摩尔纹、车轮效应

- 信号的变化太快导致采样速度跟不上，就会产生走样

### 2. Antialias

- 先做一个模糊/滤波，再采样（不能先采样再滤波）

### 3. Frequency Domain

- 傅里叶级数展开：任何一个周期函数都可以写成一系列正弦、余弦函数的线性组合以及一个常数项

- 傅里叶变换：一个函数经过一个复杂的变换，变换成了另一个函数，也可以通过逆变换变回来

- 走样：同样一种采样方法，去采样两种频率截然不同的函数，得出的结果却一样

- 滤波：把某个特定的频段删掉

- 傅里叶变换：时域\rightarrow频域,中心低频，周围高频

- 高通滤波：删掉低频段内容->提取出了轮廓
  
  - 轮廓边缘信号变化非常大，意味着高频信息

- 低通滤波：删掉高频段内容->边界模糊

### 4. Filtering = Convolution (卷积)(=Averaging)

- 时域上的卷积对应频域上的乘积

- 卷积核也可以做傅里叶变换，这样的话频域图乘以卷积核的频域图得到的新图再逆变换回去得到的结果==时域图直接乘卷积核得到的结果

- 盒式滤波（如3×3），盒子越大，越模糊，盒子越小，说明删掉的频段越少，更多的频率留了下来

### 5. Sampling=Repeating Frequency Contents

- 给一个原始的信号，乘上一个冲击函数，就可以得到采样结果；在频域上，频谱函数卷积冲击函数（傅里叶变换后的冲击函数），得到采样结果（就是重复一个原始信号的频谱）

- 走样：因为采样的不同间隔会引起频谱以另外一个不同的间隔进行移动，频谱在复制粘贴搬移的过程中发生了混合/混叠

### 6. Antialiasing

- 1：提高分辨率（但不属于反走样）

- 2：先拿掉高频信息，再采样

### 7. Antialiasing by supersampling(MSAA:Multi Sample Anti Aliasing)

- 反走样的近似，一个像素被划分为多个区域，比如4*4，根据一个像素内区域覆盖率来模糊

- 并没有提高分辨率

- 牺牲了什么：4*4的话就是16倍的计算量

- 工业上并没有划分成规则的区域，而是一些特殊图案的采样点

- 里程碑：
  
  - FXAA（Fast Approximate AA）快速近似抗锯齿
  
  - TAA（Temporal AA）时间的抗锯齿（利用之前帧）

- 超分辨率/超采样
  
  - 解决样本不足
  
  - DLSS（Deep Learning Super Sampling)
