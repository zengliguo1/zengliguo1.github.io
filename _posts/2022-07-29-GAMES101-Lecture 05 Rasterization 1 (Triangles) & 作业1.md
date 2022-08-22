---
title: GAMES101-Lecture 05 Rasterization 1 (Triangles) & 作业1
date: 2022-07-29 21:46:00 +0800
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

### 1. Perspective Projection

- aspect ratio = width/height宽高比(r就是right，t就是top)
  
  $$
  aspect=\frac{r}{t}
  $$

- field-of-view(fovY)垂直可视角度(t就是top，n就是near，因为朝-z方向，所以取个绝对值)
  
  $$
  \tan\frac{fovY}{2} = \frac{t}{\lvert n \rvert}
  $$

### 2. Canonical Cube to Screen

- 屏幕
  
  - 像素的二维数组
  
  - 数组的大小就是分辨率
  
  - 典型的光栅显示

- 变换xy平面：$[-1,1]^2到[0,width]\times[0,height]$

- 视口变换：先缩放，再把中心从原点移到$(\frac{width}{2},\frac{height}{2})$
  
  $$
  M_{viewport}=
\left(
\begin{matrix}
\frac{width}{2} & 0 & 0 & \frac{width}{2}\\
0 & \frac{height}{2} & 0 & \frac{height}{2}\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
\end{matrix}
\right)
  $$

### 3. Frame Buffer: Memory for a Raster Display

- 图像会存在显存中的不同区域中，告诉显示器显示哪一副图

### 4. LCD(Liquid Crystal Display) Pixel(液晶显示)

### 5. LED Array Display 发光二极管

### 6. Triangles-Fundamental Shape Primitives

- 三角形：
  
  - 最基础的多边形
  
  - 其他多边形可以拆成三角形
  
  - 独特的性质：
    
    - 一定是平面
    
    - 三角形内外定义清晰
    
    - 便于渐变

### 7. Sampling

- 判断像素的中心是否在三角形内
  
  ```cpp
  for(int x = 0; x < xmax; ++x)
      for(int y = 0; y < ymax; ++y)
          image[x][y] = inside(tri, x+0.5, y+0.5);
  ```

- 已知Q点和三角形P_0P_1P_2，分成三个向量：$\vec{P_0P_1}$、$\vec{P_1P_2}$、$\vec{P_2P_0}$，分别和$\vec{P_0Q}$、$\vec{P_1Q}$、$\vec{P_2Q}$做叉积，如果得到的向量方向一致，说明点Q在三条边的同侧（如都在边的左侧），那么就在三角形内，否则（一旦出现叉积结果方向不一致），就说明在三角形外。

- 边界情况：要么不做处理，要么特殊处理。

- 使用包围盒检测三角形附近的点

- Aliasing走样：Jaggies锯齿

### 作业1

- 任务描述：本次作业的任务是填写一个旋转矩阵和一个透视投影矩阵。给定三维下三个
  
  点 v0(2.0, 0.0, −2.0), v1(0.0, 2.0, −2.0), v2(−2.0, 0.0, −2.0), 你需要将这三个点的坐
  
  标变换为屏幕坐标并在屏幕上绘制出对应的线框三角形 (在代码框架中，我们已
  
  经提供了 draw_triangle 函数，所以你只需要去构建变换矩阵即可)。

- 其实不难，就是写下模型矩阵（旋转矩阵）和透视投影矩阵，但我遇到了很多很多问题，最后不得已去搜索了，查出很多错，实在惭愧，先贴代码吧，之后写下注意事项

- `get_model_matrix`函数：
  
  这个函数其实就是绕Z轴旋转的矩阵，需要注意的是cos里面的参数是弧度值，要把度数转换成弧度值才行
  
  ```cpp
  Eigen::Matrix4f get_model_matrix(float rotation_angle)
  {
      Eigen::Matrix4f model = Eigen::Matrix4f::Identity();
  
      // TODO: Implement this function
      // Create the model matrix for rotating the triangle around the Z axis.
      // Then return it.
      Eigen::Matrix4f translate;
      translate << std::cos(rotation_angle / 180.0 * MY_PI), -std::sin(rotation_angle / 180.0 * MY_PI), 0.0, 0.0,
          std::sin(rotation_angle / 180.0 * MY_PI), std::cos(rotation_angle / 180.0 * MY_PI), 0.0, 0.0,
          0.0, 0.0, 1.0, 0.0,
          0.0, 0.0, 0.0, 1.0;
      model = translate * model;
      return model;
  }
  ```

- `get_projection_matrix`函数：
  
  我在写这个函数的时候，漏洞百出。
  
  - 首先，最最要命的错误就是矩阵公式不对，我记的笔记中，M1矩阵和M2矩阵的1记错位置了，这种问题真要我自己找根本找不到，这种错误太致命了。
  
  - 其次是哪里出错呢，就是一些小地方，注释也都提到了，数行和列数错了，这种错误让我梦回中学时代。
  
  - 还有就是，明明前面算cos的时候还记得弧度值，到后面算tan的时候就忘了。

```cpp
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio,
    float zNear, float zFar)
{
    // Students will implement this function

    Eigen::Matrix4f projection = Eigen::Matrix4f::Identity();

    // TODO: Implement this function
    // Create the projection matrix for the given parameters.
    // Then return it.
    Eigen::Matrix4f orthographicM1 = Eigen::Matrix4f::Zero();
    Eigen::Matrix4f orthographicM2 = Eigen::Matrix4f::Identity();
    Eigen::Matrix4f orthographic;
    Eigen::Matrix4f persp2ortho = Eigen::Matrix4f::Zero();
    float t = std::abs(zNear) * tan(eye_fov / 2 / 180 * MY_PI);//没有转换成弧度值
    float r = aspect_ratio * t;
    //M1:
    orthographicM1(0, 0) = 1 / r; //2 / r - l;
    orthographicM1(1, 1) = 1 / t;
    orthographicM1(2, 2) = 2 / std::abs(zNear - zFar);//一开始写成了1/...
    orthographicM1(3, 3) = 1.0;//这里一开始写成了3，2

    //M2:
    //orthographicM2.topLeftCorner(3, 3).setIdentity();//这里一开始写成了setZero
    //orthographicM2(3, 3) = 1.0;//这里一开始写成了3，2
    orthographicM2(2, 3) = -(zNear + zFar) / 2;//一开始写成了0，3
    //persp2ortho:
    persp2ortho(0, 0) = zNear;
    persp2ortho(1, 1) = zNear;
    persp2ortho(3, 2) = 1.0;//一开始写成了2，3
    persp2ortho(2, 2) = zNear + zFar;
    persp2ortho(2, 3) = -zNear * zFar;

    orthographic = orthographicM1 * orthographicM2;
    projection = orthographic * persp2ortho;

    return projection;
}
```

- 结果：
  
  [![vPL6L4.jpg](https://s1.ax1x.com/2022/07/29/vPL6L4.jpg)](https://imgtu.com/i/vPL6L4)

- 有一个问题就是，在main函数中`r.set_projection(get_projection_matrix(45, 1, -0.1, -50));`，函数传入的zNear和zFar要改成-的，不然会显示反着。

- 有个问题我还是不太懂，就是view矩阵，他其实就是做了个平移，相当于是z减少了5。问题就是相机的初始位置到底在哪里呢？很奇怪，是（0,0）吗，相机的注视方向呢？是沿着Z轴吗？相机的头顶方向呢？其实只要矩阵写对了，就能看到三角形，也许这道作业就是为了让我们了解下model变换和透视投影变换吧，其他的问题可能会在后面解决吧。
