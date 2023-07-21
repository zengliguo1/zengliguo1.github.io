---
title: GAMES101-Lecture 06 Rasterization 2(Antialiasing and Z-Buffering)
date: 2022-08-05 20:59:00 +0800
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

## Sample Artifacts(Errors,mistakes,inaccuracies图形学黑话)

- 锯齿、摩尔纹、车轮效应

- 信号的变化太快导致采样速度跟不上，就会产生走样

[![p9gvPXT.jpg](https://s1.ax1x.com/2023/05/15/p9gvPXT.jpg)](https://imgse.com/i/p9gvPXT)

### Antialias反走样

[![p9gjSsO.jpg](https://s1.ax1x.com/2023/05/15/p9gjSsO.jpg)](https://imgse.com/i/p9gjSsO)

- 先做一个模糊/滤波，再采样（不能先采样再滤波）
- 需要先模糊再采样，不能反过来。那么为什么不能反过来呢，那是因为先采样会发生混叠，如果要完全删掉混叠就要删掉大量信号，这样显然是不对的方法（原因放在后面看，后面会说什么是混叠）

## Frequency Domain

- 傅里叶级数展开：任何一个周期函数都可以写成一系列正弦、余弦函数的线性组合以及一个常数项

[![p9gj4kd.jpg](https://s1.ax1x.com/2023/05/15/p9gj4kd.jpg)](https://imgse.com/i/p9gj4kd)

- 傅里叶变换：一个函数经过一个复杂的变换，变换成了另一个函数，也可以通过逆变换变回来

[![p9gjH6f.jpg](https://s1.ax1x.com/2023/05/15/p9gjH6f.jpg)](https://imgse.com/i/p9gjH6f)

- 走样：同样一种采样方法，去采样两种频率截然不同的函数，得出的结果却一样，无法区分

- 滤波：把某个特定的频段删掉

- 傅里叶变换：时域->频域,中心低频，周围高频(越往外，频率越高，图像的频率是表征图像中灰度变化剧烈程度的指标)

- 白色代表着在该频段信号较多，比如中心很多白色说明图像的低频信号较多

- 为什么下面右侧的频域图会有横竖两条白线，那是因为傅里叶变换要变换周期函数，而这张图并不是，所以就当成在边缘会再次重复这张图，横竖都会这样重复，所以可以忽略这两条线

[![p9gzQTe.jpg](https://s1.ax1x.com/2023/05/15/p9gzQTe.jpg)](https://imgse.com/i/p9gzQTe)

- 高通滤波：删掉低频段内容->提取出了轮廓

[![p9gz5tJ.jpg](https://s1.ax1x.com/2023/05/15/p9gz5tJ.jpg)](https://imgse.com/i/p9gz5tJ)

- 轮廓边缘信号变化非常大，意味着高频信息

- 低通滤波：删掉高频段内容->边界模糊

[![p92SFnf.jpg](https://s1.ax1x.com/2023/05/15/p92SFnf.jpg)](https://imgse.com/i/p92SFnf)

- 带通滤波：去掉高频和低频，留某一段

[![p92SAHS.jpg](https://s1.ax1x.com/2023/05/15/p92SAHS.jpg)](https://imgse.com/i/p92SAHS)

### Filtering = Convolution (卷积)(=Averaging)

[![p9298fJ.jpg](https://s1.ax1x.com/2023/05/15/p9298fJ.jpg)](https://imgse.com/i/p9298fJ)

- **时域上的卷积对应频域上的乘积，时域上的乘积对应频域上的卷积**

[![p929am6.jpg](https://s1.ax1x.com/2023/05/15/p929am6.jpg)](https://imgse.com/i/p929am6)

- 卷积核也可以做傅里叶变换，这样的话频域图乘以卷积核的频域图得到的新图再逆变换回去得到的结果==时域图直接乘卷积核得到的结果

- 盒式滤波（如3×3），盒子越大（对应的频域图中，低频变多，高频变少，中心白色面积越小），越模糊，盒子越小，说明删掉的频段越少，更多的频率留了下来

### Sampling=Repeating Frequency Contents

[![p92C3Hf.jpg](https://s1.ax1x.com/2023/05/15/p92C3Hf.jpg)](https://imgse.com/i/p92C3Hf)

- 给一个原始的信号，乘上一个冲击函数，就可以得到采样结果；在频域上，频谱函数卷积冲击函数（傅里叶变换后的冲击函数），得到采样结果（采样就是重复一个原始信号的频谱）

- 如果采样速度不够大，也就是采样间隔越大，频谱复制的间隔就会变小造成混叠(至于为什么会一个变大一个变小，我目前还不是很清楚)

[![p92CL2d.jpg](https://s1.ax1x.com/2023/05/15/p92CL2d.jpg)](https://imgse.com/i/p92CL2d)

- 走样：因为采样的不同间隔会引起频谱以另外一个不同的间隔进行移动，频谱在复制粘贴搬移的过程中发生了混合/混叠

### Antialiasing

- 1：提高分辨率（但不属于反走样）

- 2：先拿掉高频信息，再采样

[![p92P8MR.jpg](https://s1.ax1x.com/2023/05/15/p92P8MR.jpg)](https://imgse.com/i/p92P8MR)

* 那么如何模糊？使用低通滤波器来卷积（也可以说是求个平均）

[![p92Pgdf.jpg](https://s1.ax1x.com/2023/05/15/p92Pgdf.jpg)](https://imgse.com/i/p92Pgdf)

### Antialiasing by supersampling(MSAA:Multi Sample Anti Aliasing)

- 反走样的近似，一个像素被划分为多个区域，比如4*4，根据一个像素内区域覆盖率来模糊

- MSAA只是做到了模糊，并没有提高屏幕的分辨率

- 牺牲了什么：4*4的话就是16倍的计算量

- 工业上并没有划分成规则的区域，而是一些特殊图案的采样点

- 里程碑：
  
  - FXAA（Fast Approximate AA）快速近似抗锯齿（与采样无关，是先算出有锯齿的图，再想办法把锯齿换掉）
  
  - TAA（Temporal AA）时间的抗锯齿（利用之前帧）

- 超分辨率/超采样
  
  - 解决样本不足
  
  - DLSS（Deep Learning Super Sampling)
