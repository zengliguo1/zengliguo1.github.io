---
title: GAMES101-Lecture 09 Shading3(Texture Mapping cont.)
date: 2022-08-25 23:00:00 +0800
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

### 1. Interpolation Across Triangles: Barycentric Coordinates(重心坐标)

- 为了获得三角形内值的平滑过渡

- 纹理坐标、颜色、法线的插值

- Barycentric Coordinates：重心坐标是定义在一个三角形上的$(\alpha, \beta , \gamma)$

$$
(x,y)=\alpha A+\beta B + \gamma C
\\
\alpha+ \beta  + \gamma =1
$$

- 三角形是二维的，所以用两个数就可以表示，因此三个未知数知道两个即可

- 如果点在三角形内部，那么$\alpha、 \beta 、\gamma$都是非负的

- A点的重心坐标$(\alpha, \beta ,\gamma)=(1,0,0)$就是A点自己

- 计算任何一点的重心坐标：（$A_A、B_B、C_C$代表的是面积）

- [![p9IxEi6.jpg](https://s1.ax1x.com/2023/05/22/p9IxEi6.jpg)](https://imgse.com/i/p9IxEi6)

- $$
  \alpha = \frac{A_A}{A_A+B_B+C_C}
\\
\beta = \frac{B_B}{A_A+B_B+C_C}\\
\gamma = \frac{C_C}{A_A+B_B+C_C}
  $$

- $(\alpha, \beta ,\gamma)=(1/3,1/3,1/3)$是三角形的重心

- [![p9IxROJ.jpg](https://s1.ax1x.com/2023/05/22/p9IxROJ.jpg)](https://imgse.com/i/p9IxROJ)

- 计算每个点的重心坐标的一般公式：(懒得手打了，免得敲半天再像作业1时候那样敲错)
  
  [![v2KBIxjpg](https://s1.ax1x.com/2022/08/25/v2KBIx.jpg)](https://imgse.com/i/v2KBIx)

- 使用重心坐标：
  
  $$
  V=\alpha V_A + \beta V_B+\gamma V_C
  $$

- V可以是位置、纹理坐标、颜色、法线、深度、材质属性

- 缺点：投影前后对同一个三角形内得同一个点求重心坐标很有可能就不同了

- 所以如果是三维空间内，那就求三维空间内的重心坐标：屏幕上的一个像素**逆变换**回去，到三维空间中，求出该像素的重心坐标，再插值

### 2. Applying Textrue

- 已知的是三角形顶点的uv，通过重心坐标就可以求得三角形内任意一点的uv，就可以获得uv图的信息了

- 可以认为，纹理定义的就是漫反射系数

### 3. Texture Magnification(纹理的放大)

[![p9ok260.jpg](https://s1.ax1x.com/2023/05/22/p9ok260.jpg)](https://imgse.com/i/p9ok260)

- 纹理图相较于物体太小，很高清的墙，贴了一张很低清的图

- a pixel on a texture - a texel(纹理元素)

- **Bilinear interpolation** 双线性插值
  
  [![v2MttPjpg](https://s1.ax1x.com/2022/08/25/v2MttP.jpg)](https://imgse.com/i/v2MttP)

- 先横向插值得到u0和u1，再纵向插值，得到最终的点的颜色值，我认为可以根据分割的四个矩形的面积来分配四个点对目标点的贡献

- 缺点：质量还是差一些

- **Bicubic interpolation**双三次插值：取一个点临近16个点，做插值，每四个做三次的插值（不是线性）

### 4. Texture Magnification(hard case)

[![p9okRXV.jpg](https://s1.ax1x.com/2023/05/22/p9okRXV.jpg)](https://imgse.com/i/p9okRXV)

- 纹理相对物体过大

- 走样

- 屏幕上的像素覆盖的纹理上的区域大小是不相同的（近处的像素覆盖小，也就是说物体对于纹素较小，取最近的点就会产生锯齿；而远处的像素覆盖大，也就是物体对于文素来说较大，需要取平均而产生摩尔纹）

- [![p9okHpR.jpg](https://s1.ax1x.com/2023/05/22/p9okHpR.jpg)](https://imgse.com/i/p9okHpR)

- 解决：
  
  - 超采样：开销太大
    
    [![p9oAphd.jpg](https://s1.ax1x.com/2023/05/22/p9oAphd.jpg)](https://imgse.com/i/p9oAphd)
  
  - 避免采样：Range Query（范围查询）

- **Mipmap** ：允许（快速、近似、方形）的范围查询
  
  - 在渲染之前把一张纹理的mipmap都生成
    
    [![v2QlCVjpg](https://s1.ax1x.com/2022/08/25/v2QlCV.jpg)](https://imgse.com/i/v2QlCV)

- 只是多了1/3的存储量

- 可以这么想，把第0层复制三份，放在左上右上和左下，接着把第1层放在右下角方块的左上右上和左下，然后递归，最后会发现，第一层和往上的所有层复制三份后刚好是一份第0层，所以就是1/3

- [![p9oEadS.jpg](https://s1.ax1x.com/2023/05/22/p9oEadS.jpg)](https://imgse.com/i/p9oEadS)

- **ComPuting Mipmap Level D:**
  
  [![v2liZ9.jpg](https://s1.ax1x.com/2022/08/25/v2liZ9.jpg)](https://imgse.com/i/v2liZ9)

- 图解：某个像素取其上面和右面的像素，映射到uv图（先从投影也就是光栅化后的像素映射回三维空间中的点（使用逆变换），插值得到uv的坐标），计算在uv图上，两点距离中心点的距离（其实是微分，理解成中心点移动到右边或者上边点的距离），L1和L2，L取二者较大值，以L为近似方形的边长，$D=log_2L$。如果区域为1\*1，那么D=0，说明就在原来的纹理图上，如果区域为4\*4，那么D=2，说明在第2张纹理图（32\*32）上，该区域代表这张纹理图的一个像素

- mipmap缺点是不是渐变的，层数是离散的，所以怎么办呢，插值！

- **Trilinear Interpolation三线性插值**查第1.8层（带小数）的，先分别在第1层和第2层分别在自己对应的uv图上通过双线性插值算一个值，再把计算得来的两个值做一次线性插值。在游戏的实时渲染中应用很广泛。
  
  [![p9outLn.jpg](https://s1.ax1x.com/2023/05/22/p9outLn.jpg)](https://imgse.com/i/p9outLn)

- 在远处会发现糊掉了，叫做overblur，过分模糊，如上图
  
  - 原因：正方形区域导致
    
    [![p9ouoSe.jpg](https://s1.ax1x.com/2023/05/22/p9ouoSe.jpg)](https://imgse.com/i/p9ouoSe)
  
  - 解决：Anisotropic Filtering 各向异性过滤，这样就可以近似成长方形，可以在一个长条形区域快速查询(Ripmap)。但仍然存在问题：就是如果是个倾斜的矩形，还是不好框。而且除此之外，开销增加了3倍。还会选择几x，选择2x就是算到第二层
  
  - EWA filtering : 拆成很多不同的圆形去覆盖一个不规则的矩形（多次查询），代价开销大
  
  - [![p9oKplQ.jpg](https://s1.ax1x.com/2023/05/22/p9oKplQ.jpg)](https://imgse.com/i/p9oKplQ)

### 作业3

* 作业描述：
  
      在这次编程任务中，我们会进一步模拟现代图形技术。我们在代码中添加了
  
  Object Loader(用于加载三维模型), Vertex Shader 与 Fragment Shader，并且支持
  
  了纹理映射。
  
      而在本次实验中，你需要完成的任务是:
  
  1. 修改函数 rasterize_triangle(const Triangle& t) in rasterizer.cpp: 在此
  
  处实现与作业 2 类似的插值算法，实现法向量、颜色、纹理颜色的插值。
  
  2. 修改函数 get_projection_matrix() in main.cpp: 将你自己在之前的实验中
  
  实现的投影矩阵填到此处，此时你可以运行 ./Rasterizer output.png normal
  
  来观察法向量实现结果。
  
  3. 修改函数 phong_fragment_shader() in main.cpp: 实现 Blinn-Phong 模型计
  
  算 Fragment Color.
  
  4. 修改函数 texture_fragment_shader() in main.cpp: 在实现 Blinn-Phong
  
  的基础上，将纹理颜色视为公式中的 kd，实现 Texture Shading Fragment
  
  Shader.
  
  5. 修改函数 bump_fragment_shader() in main.cpp: 在实现 Blinn-Phong 的
  
  基础上，仔细阅读该函数中的注释，实现 Bump mapping.
  
  6. 修改函数 displacement_fragment_shader() in main.cpp: 在实现 Bump
  
  mapping 的基础上，实现 displacement mapping

* 上来报错namespace "std" has no member "optional"，这是因为optional是C++17新特性，我的vs默认是C++14，在项目属性->常规中修改标准版本为C++17即可
