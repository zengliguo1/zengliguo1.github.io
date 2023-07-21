---
title: GAMES101-Lecture 14 Ray Tracing 2
date: 2023-07-07 14:32:00 +0800
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

## Using AABBs to accelerate ray tracing

### Uniform grids

* 主旨：多做光线与盒子求交

* 先把场景的BB框住，然后在BB里创建格子

* 记录每个有物体（表面）存在的格子(下图中右上角少画了一个格子)

[![pCcSay4.jpg](https://s1.ax1x.com/2023/07/07/pCcSay4.jpg)](https://imgse.com/i/pCcSay4)

* 判断跟光线相交的格子里有没有物体

* 如果有物体，那么说明光线可能和该物体相交，就计算交点

* 在光线的发射过程中（对于图中的光线），只需判断当前格子的上方和右方是否为光线下一个相交的格子

[![pCcpPpT.jpg](https://s1.ax1x.com/2023/07/07/pCcpPpT.jpg)](https://imgse.com/i/pCcpPpT)

### Grid Resolution

* 不能太稀疏（比如1\*1\*1的格子，相当于没做BBAA），也不能太密集，不然和格子也求交太多次，所以要有一个均衡

[![pCcplcD.jpg](https://s1.ax1x.com/2023/07/07/pCcplcD.jpg)](https://imgse.com/i/pCcplcD)

* BBAA适合比较均匀的场景，不适合有大片空区域的场景，会产生“Teapot in a stadium”问题

### Spatial partitions(空间划分)

* 基于格子的不足之处进行优化，在物体分布稀疏的地方格子大一点，，分布密集的地方格子小一点

* 八叉树（Oct-Tree），对空间切三刀，然后如果某个小空间中物体比较多，继续切，直到空间内没物体或者物体数量差不多了，但维度再高的话，就是$2^n$叉树，这显然不好（我认为老师意思是说这么切下去还是切太多了），为解决这个问题，KD-Tree应运而生

* KD-Tree：每次砍一刀，然后再根据情况在分出的一个空间内再砍一刀，这样就类似二叉树，每次水平一刀，竖直一刀，这样划分的相对均匀一点

* BSP-Tree：不是横平竖直地砍，所以比较复杂

[![pCcF9Qf.jpg](https://s1.ax1x.com/2023/07/07/pCcF9Qf.jpg)](https://imgse.com/i/pCcF9Qf)

### KD-Tree Pre-Processing

* 在做光追之前提前划分好，在中间节点记录它之后划分了什么样的格子，在叶子节点来实际存储和格子相交的物体（三角形）

[![pCcFdOO.jpg](https://s1.ax1x.com/2023/07/07/pCcFdOO.jpg)](https://imgse.com/i/pCcFdOO)

* 首先，光线和整个BBAA*（A)有交点，那么对于子节点（也就是1和B）都判断是否有交点

* 都相交，但1也就到此为止，去计算1中所有物体是否与光线相交，然后去计算B的子节点2和C

* 还是都相交，但2也到此为止，去计算2中所有物体是否与光线相交，然后去计算C的子节点3和D

* 还是都相交，继续算

* 到4和5，其中5是没有交点的，所以就不计算了

* 有几个问题：
  
  * 如何判断包围盒的格子到底和哪些三角形相交，这个问题比较困难处理
  
  * 一个物体与多个包围盒相交，也就是多个叶子节点都要储存该物体

### Object Partitions & Bounding Volume Hierarchy(BVH)

* 解决了KD-Tree的两个问题，既不用求交，而且一个三角形不会与多个包围盒相交，只会存在一个包围盒中

* 但问题是包围盒有可能相交，怎么划分很讲究

[![pCc3MX4.jpg](https://s1.ax1x.com/2023/07/07/pCc3MX4.jpg)](https://imgse.com/i/pCc3MX4)

* 怎样区分一个节点：
  
  * 选择一个维度去划分（x、y、z轴）
  
  * 总是沿着最长的轴去切
  
  * 区分节点（切）的时候，从中间物体的位置切，快速找出中位数，使用快速选择算法，只需要O(n)的复杂度

* 如果是动态场景，就需要重新计算BVH了

* BVH伪代码：

[![pCc8pU1.jpg](https://s1.ax1x.com/2023/07/07/pCc8pU1.jpg)](https://imgse.com/i/pCc8pU1)

### Spatial VS Object Partitions

* 空间划分和物体划分的区别

* 空间：
  
  * 划分了不重叠的空间区域
  
  * 一个物体可能会被包含在多个空间中

* 物体：
  
  * 物体被划分到不同的子集中
  
  * 包围盒可能会重叠

## Basic radiometry (辐射度量学)

### Motivation

* 为光赋予物理意义

* 定义了一些列的方法和单位来描述光照

* 准确计量光在空间中的各种属性
  
  * Radiant flux：光通量，单位时间内通过的光总量，单位watt或者lumen（lm）
  
  * intensity：光强，单位立方角光通量，单位Watt/steradians，也就是candelas
  
  * irradiance：辉度，单位面积光通量，瓦特/平方米
  
  * radiance：光亮度

### Radiant Energy and Flux(Power)

[![pCcJ91K.jpg](https://s1.ax1x.com/2023/07/07/pCcJ91K.jpg)](https://imgse.com/i/pCcJ91K)

* Radiant Energy：可以理解为光的能量，单位是焦耳

* Radiant Flux(power)：单位时间的能量，单位是瓦特

* Flux（另外一种解释）：光打到一个感光的平面，单位时间内通过的光子的数量

[![pCcJC6O.jpg](https://s1.ax1x.com/2023/07/07/pCcJC6O.jpg)](https://imgse.com/i/pCcJC6O)

### Important Light Measurements of Interest

[![pCcJRgK.jpg](https://s1.ax1x.com/2023/07/07/pCcJRgK.jpg)](https://imgse.com/i/pCcJRgK)

* Radiant Intensity：光源散发的光

* irradiance：一个平面接收的光

* radiance：在传播中的光

### Radiant Intensity

* 单位立体角的能量

[![pCcJ4De.jpg](https://s1.ax1x.com/2023/07/07/pCcJ4De.jpg)](https://imgse.com/i/pCcJ4De)

* 二维中角度一般指弧度，单位是radians

* 三维中立体角：球上一块面积除以半径的平方，球有$4\pi$的立体角，单位是steradians

[![pCcYZb4.jpg](https://s1.ax1x.com/2023/07/07/pCcYZb4.jpg)](https://imgse.com/i/pCcYZb4)

* 单位立体角/微分立体角：微分面积/半径平方

[![pCcYfGq.jpg](https://s1.ax1x.com/2023/07/07/pCcYfGq.jpg)](https://imgse.com/i/pCcYfGq)

* 之后会用单位立体角w来表示一个单位向量

[![pCcY4zV.jpg](https://s1.ax1x.com/2023/07/07/pCcY4zV.jpg)](https://imgse.com/i/pCcY4zV)

* Radiant Intensity就是单位立体角的能量

[![pCcYIMT.jpg](https://s1.ax1x.com/2023/07/07/pCcYIMT.jpg)](https://imgse.com/i/pCcYIMT)

## 作业6

    在之前的编程练习中，我们实现了基础的光线追踪算法，具体而言是光线传

输、光线与三角形求交。我们采用了这样的方法寻找光线与场景的交点：遍历场景

中的所有物体，判断光线是否与它相交。在场景中的物体数量不大时，该做法可以

取得良好的结果，但当物体数量增多、模型变得更加复杂，该做法将会变得非常低

效。因此，我们需要加速结构来加速求交过程。在本次练习中，我们重点关注物体

划分算法 Bounding Volume Hierarchy (BVH)。本练习要求你实现 Ray-Bounding

Volume 求交与 BVH 查找。

    首先，你需要从上一次编程练习中引用以下函数：

* Render() in Renderer.cpp: 将你的光线生成过程粘贴到此处，并且按照新框架更新相应调用的格式。

* Triangle::getIntersection in Triangle.hpp: 将你的光线-三角形相交函数粘贴到此处，并且按照新框架更新相应相交信息的格式。

    在本次编程练习中，你需要实现以下函数：

* IntersectP(const Ray& ray, const Vector3f& invDir,const std::array\<int, 3>& dirIsNeg) in the Bounds3.hpp: 这个函数的作用是判断包围盒 BoundingBox 与光线是否相交，你需要按照课程介绍的算法实现求交过程。

* getIntersection(BVHBuildNode* node, const Ray ray)in BVH.cpp: 建立 BVH 之后，我们可以用它加速求交过程。该过程递归进行，你将在其中调用你实现的 Bounds3::IntersectP

### Render()函数

* 其实render函数已经写的差不多了，计算投射的光线已经算出来了，只需要再写一行代码调用函数即可，需要调用的是Scene类的castRay函数，与上一次作业不同，这次作业的形参做了修改，需要直接传入ray对象，至于深度depth，不用管，就是记录个射线的弹射次数，填0即可

* `framebuffer[m++] = scene.castRay(ray, 0);`，加这行代码即可，完整代码如下：

```cpp
void Renderer::Render(const Scene& scene)
{
    std::vector<Vector3f> framebuffer(scene.width * scene.height);

    float scale = tan(deg2rad(scene.fov * 0.5));
    float imageAspectRatio = scene.width / (float)scene.height;
    Vector3f eye_pos(-1, 5, 10);
    int m = 0;
    for (uint32_t j = 0; j < scene.height; ++j) {
        for (uint32_t i = 0; i < scene.width; ++i) {
            // generate primary ray direction
            float x = (2 * (i + 0.5) / (float)scene.width - 1) *
                      imageAspectRatio * scale;
            float y = (1 - 2 * (j + 0.5) / (float)scene.height) * scale;
            // TODO: Find the x and y positions of the current pixel to get the
            // direction
            //  vector that passes through it.
            // Also, don't forget to multiply both of them with the variable
            // *scale*, and x (horizontal) variable with the *imageAspectRatio*

            // Don't forget to normalize this direction!

            Vector3f dir = Vector3f(x, y, -1); // Don't forget to normalize this direction!
            dir = normalize(dir);
            Ray ray(eye_pos, dir);
            framebuffer[m++] = scene.castRay(ray, 0);
        }
        UpdateProgress(j / (float)scene.height);
    }
    UpdateProgress(1.f);

    // save framebuffer to file
    FILE* fp = fopen("binary.ppm", "wb");
    (void)fprintf(fp, "P6\n%d %d\n255\n", scene.width, scene.height);
    for (auto i = 0; i < scene.height * scene.width; ++i) {
        static unsigned char color[3];
        color[0] = (unsigned char)(255 * clamp(0, 1, framebuffer[i].x));
        color[1] = (unsigned char)(255 * clamp(0, 1, framebuffer[i].y));
        color[2] = (unsigned char)(255 * clamp(0, 1, framebuffer[i].z));
        fwrite(color, 1, 3, fp);
    }
    fclose(fp);    
}
```

### Triangle::getIntersection函数

* 这个部分说难吧也不难，说简单吧，倒也不好想，基本的算法已经是算完了，主要就是把算好的内容返回，那么返回的是一个Intersection对象，所以就需要了解下Intersection类，需要根据这个函数的计算结果来创建好要返回的Intersection对象，完整代码如下：

```cpp
inline Intersection Triangle::getIntersection(Ray ray)
{
    Intersection inter;

    if (dotProduct(ray.direction, normal) > 0)
        return inter;
    double u, v, t_tmp = 0;
    //下面的代码已经是Moller Trumbore Algorithm了
    Vector3f pvec = crossProduct(ray.direction, e2);//该向量与三角形法线正交
    double det = dotProduct(e1, pvec);//det 的几何意义是三角形两个边向量以及光线方向向量构成的三维矩阵的行列式。
    if (fabs(det) < EPSILON)//如果det接近0，说明两个边向量接近平行,从而导致三角形面积接近0,即三角形退化为线段或点
        return inter;

    double det_inv = 1. / det;
    Vector3f tvec = ray.origin - v0;
    u = dotProduct(tvec, pvec) * det_inv;
    if (u < 0 || u > 1)
        return inter;
    Vector3f qvec = crossProduct(tvec, e1);
    v = dotProduct(ray.direction, qvec) * det_inv;
    if (v < 0 || u + v > 1)
        return inter;
    t_tmp = dotProduct(e2, qvec) * det_inv;

    // TODO find ray triangle intersection
    if (t_tmp < 0) return inter;

    inter.happened = true;//是否相交
    inter.coords = ray(t_tmp);//交点坐标
    inter.normal = normal;//交点处三角形法线
    inter.distance = t_tmp;//d
    inter.obj = this;//三角形
    inter.m = m;//交点处三角形的材质

    return inter;
}
```

### IntersectP函数

* 其实这个就按课程上讲的过程来算就行了，算出光线进入时间和推出时间即可，我看有的博主使用vector相乘，规避了for循环，这点还是很不错的。有一个点让人比较费解，就是IntersectP函数的形参中有一个是`const std::array<int, 3>& dirIsNeg` ，实际上我全程没用到这个函数。我的代码如下：

```cpp
inline bool Bounds3::IntersectP(const Ray& ray, const Vector3f& invDir,
                                const std::array<int, 3>& dirIsNeg) const
{
    // invDir: ray direction(x,y,z), invDir=(1.0/x,1.0/y,1.0/z), use this because Multiply is faster that Division
    // dirIsNeg: ray direction(x,y,z), dirIsNeg=[int(x>0),int(y>0),int(z>0)], use this to simplify your logic
    // TODO test if ray bound intersects
    double t_enter, t_exit;
    double t_min[3]{}, t_max[3]{};
    for (int i = 0; i < 3; i++)
    {
        t_min[i] = std::min((pMin[i] - ray.origin[i]) * invDir[i], (pMax[i] - ray.origin[i]) * invDir[i]);
        t_max[i] = std::max((pMin[i] - ray.origin[i]) * invDir[i], (pMax[i] - ray.origin[i]) * invDir[i]);
    }
    t_enter = std::fmax(t_min[0], std::fmax(t_min[1],t_min[2]));
    t_exit = std::fmin(t_max[0], std::fmin(t_max[1], t_max[2]));
    if (t_enter < t_exit && t_exit >= 0.0) return true;

    return false;
}
```

#### getIntersection函数

* 这个函数就照着课程ppt的伪码来做就行了，有几点让人费解，就是上面说的参数，有一个我就没用，还有就是光线方向的倒数，这个直接在ray对象那取出来就好，不用自己算，我一开始还在想怎么算，万一除以个0该怎么处理，实际上在ray的构造函数里就自己算了，而且就是简单的除以，没考虑除以0的情况，呃呃。代码如下：

```cpp
Intersection BVHAccel::getIntersection(BVHBuildNode* node, const Ray& ray) const
{
    // TODO Traverse the BVH to find intersection
    Intersection inter;
    if (!node || !node->bounds.IntersectP(ray, ray.direction_inv, { 0,0,0 }))
        return inter;
    if (node->left == nullptr && node->right == nullptr)
    {
        return node->object->getIntersection(ray);
    }
    Intersection inter1 = getIntersection(node -> left, ray);
    Intersection inter2 = getIntersection(node->right, ray);

    return inter1.distance < inter2.distance ? inter1 : inter2;

}
```

[![pCTD5PH.jpg](https://s1.ax1x.com/2023/07/18/pCTD5PH.jpg)](https://imgse.com/i/pCTD5PH)

### 提高题：SAH

* 这个sah我搞了不少时间才弄懂，但就结果而言，提升并不是很高，具体过程我就不多赘述了，我挂两个我参考的链接，我在代码处有一些注释

* [BVH with SAH (Bounding Volume Hierarchy with Surface Area Heuristic) - lookof - 博客园 (cnblogs.com)](https://www.cnblogs.com/lookof/p/3546320.html)

* [(23条消息) 光线求交加速算法：边界体积层次结构(Bounding Volume Hierarchies)2-表面积启发式法(The Surface Area Heuristic)_0小龙虾0的博客-CSDN博客](https://blog.csdn.net/qq_39300235/article/details/106999514#:~:text=%E8%A1%A8%E9%9D%A2%E7%A7%AF%E5%90%AF%E5%8F%91%E5%BC%8F%E6%B3%95%20%28The%20Surface%20Area%20Heuristic%29,%E8%AF%A5%E7%AE%97%E6%B3%95%E6%8F%90%E4%BE%9B%E4%BA%86%E6%89%8E%E5%AE%9E%E7%9A%84%E6%88%90%E6%9C%AC%E6%A8%A1%E5%9E%8B%E6%9D%A5%E4%BC%B0%E8%AE%A1%E4%BA%86%E5%93%AA%E4%B8%AA%E5%88%92%E5%88%86%E4%BD%8D%E7%BD%AE%E4%BC%9A%E5%AF%BC%E8%87%B4%E5%B0%84%E7%BA%BF%E5%92%8C%E5%9B%BE%E5%85%83%E7%9B%B8%E4%BA%A4%E6%9C%80%E4%B8%BA%E4%BE%BF%E5%AE%9C%E3%80%82.%20SAH%E6%A8%A1%E5%9E%8B%E4%BC%B0%E8%AE%A1%E6%89%A7%E8%A1%8C%E5%B0%84%E7%BA%BF%E7%9B%B8%E4%BA%A4%E6%B5%8B%E8%AF%95%E7%9A%84%E8%AE%A1%E7%AE%97%E6%88%90%E6%9C%AC%EF%BC%8C%E5%8C%85%E6%8B%AC%E9%81%8D%E5%8E%86%E6%A0%91%E7%9A%84%E8%8A%82%E7%82%B9%E6%89%80%E8%8A%B1%E8%B4%B9%E7%9A%84%E6%97%B6%E9%97%B4%EF%BC%8C%E4%BB%A5%E5%8F%8A%E9%92%88%E5%AF%B9%E7%89%B9%E5%AE%9A%E5%9B%BE%E5%85%83%E5%88%86%E5%8C%BA%E7%9A%84%E5%B0%84%E7%BA%BF%E4%B8%8E%E5%9B%BE%E5%85%83%E7%9B%B8%E4%BA%A4%E6%B5%8B%E8%AF%95%E6%89%80%E8%8A%B1%E8%B4%B9%E7%9A%84%E6%97%B6%E9%97%B4%E3%80%82.%20%E7%84%B6%E5%90%8E%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%9E%84%E5%BB%BA%E5%8A%A0%E9%80%9F%E7%BB%93%E6%9E%84%E7%9A%84%E7%AE%97%E6%B3%95%E4%BB%A5%E5%B0%86%E6%80%BB%E6%88%90%E6%9C%AC%E9%99%8D%E8%87%B3%E6%9C%80%E4%BD%8E%E4%B8%BA%E7%9B%AE%E6%A0%87%E3%80%82.%20%E9%80%9A%E5%B8%B8%EF%BC%8C%E4%BD%BF%E7%94%A8%E4%B8%80%E7%A7%8D%E8%B4%AA%E5%A9%AA%E7%AE%97%E6%B3%95%EF%BC%8C%E8%AF%A5%E7%AE%97%E6%B3%95%E5%8F%AF%E6%9C%80%E5%A4%A7%E7%A8%8B%E5%BA%A6%E5%9C%B0%E5%87%8F%E5%B0%91%E5%8D%95%E7%8B%AC%E6%9E%84%E5%BB%BA%E7%9A%84%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E4%B8%AD%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E7%9A%84%E6%88%90%E6%9C%AC%E3%80%82.%20SAH%E6%88%90%E6%9C%AC%E6%A8%A1%E5%9E%8B%E8%83%8C%E5%90%8E%E7%9A%84%E6%80%9D%E6%83%B3%E5%BE%88%E7%AE%80%E5%8D%95%EF%BC%8C%E5%AE%83%E6%9C%89%E4%B8%A4%E7%A7%8D%E6%96%B9%E6%B3%95%E8%AE%A1%E7%AE%97%E3%80%82.)

```cpp
BVHBuildNode* BVHAccel::recursiveBuild(std::vector<Object*> objects)
{
    BVHBuildNode* node = new BVHBuildNode();

    // Compute bounds of all primitives in BVH node
    Bounds3 bounds;
    for (int i = 0; i < objects.size(); ++i)
        bounds = Union(bounds, objects[i]->getBounds());
    if (objects.size() == 1) {
        // Create leaf _BVHBuildNode_
        node->bounds = objects[0]->getBounds();
        node->object = objects[0];
        node->left = nullptr;
        node->right = nullptr;
        return node;
    }
    else if (objects.size() == 2) {
        node->left = recursiveBuild(std::vector{objects[0]});
        node->right = recursiveBuild(std::vector{objects[1]});

        node->bounds = Union(node->left->bounds, node->right->bounds);
        return node;
    }
    else {
        Bounds3 centroidBounds;
        for (int i = 0; i < objects.size(); ++i)
            centroidBounds =
                Union(centroidBounds, objects[i]->getBounds().Centroid());
        int dim = centroidBounds.maxExtent();
        SplitMethod splitMethod = SplitMethod::NAIVE;

        switch (splitMethod)
        {

            case SplitMethod::SAH :
            {

                const int nBuckets = 12;
                const float SAHTravCost = 0.125f;//遍历花费的时间成本
                //const float SAHInterCost = 1.f;//检查一个物体花费的时间成本,其实可以不算
                BucketInfo buckets[nBuckets];
                //遍历三角形，查看三角形的质心位于哪一个桶中
                for (int i = 0; i < objects.size(); i++)
                {
                    //用来存当前图元也就是三角形质心位于几号桶中,dim是最松散的一个轴
                    int b = nBuckets * centroidBounds.Offset(objects[i]->getBounds().Centroid())[dim];
                    if (b == nBuckets) b = nBuckets - 1;
                    buckets[b].count++;
                    buckets[b].bounds = Union(buckets[b].bounds, objects[i]->getBounds());
                }
                //下面计算最小成本
                float cost[nBuckets - 1]{ 0.f };
                //按i的序号，将桶分成两部分，所以不取最后一个桶
                //因为如果左边取了最后一个桶，那么相当于右边没有桶了，这也是为啥cost数组长度为nBuckets - 1
                for (int i = 0; i < nBuckets - 1; i++)
                {
                    Bounds3 b0, b1;
                    int cnt0 = 0, cnt1 = 0;
                    for (int j = 0; j <= i; j++)
                    {
                        b0 = Union(b0, buckets[j].bounds);
                        cnt0++;
                    }
                    for (int j = i + 1; j <= nBuckets - 1; j++)
                    {
                        b1 = Union(b1, buckets[j].bounds);
                        cnt1++;
                    }
                    cost[i] = SAHTravCost + (cnt0 * b0.SurfaceArea()  + cnt1 * b1.SurfaceArea() ) / bounds.SurfaceArea();
                }
                //找出最小时间成本的划分桶序号
                //船新的找最小值下标方法，一行代码解决
                int minCostSplitBucket = std::distance(cost, std::min_element(cost, cost + nBuckets - 1));
                //下面是传统的找最小值方法
                float minCost = cost[0];
                for (int i = 1; i < nBuckets - 1; i++)
                {
                    if (cost[i] < minCost)
                    {
                        minCost = cost[i];
                        minCostSplitBucket = i;
                    }
                }
                float leafCost = objects.size();//直接将当前所有图元也就是三角形都检测一遍的时间成本
                //如果当前图元数量大于一个节点允许的最大图元数量，那就需要按桶序号来划分两个区域
                //或者使用区域划分的时间成本确实小于将当前图元不划分的时间成本，那也要划分
                //但这个maxPrimsInNode设置为1.所以都会执行这段代码，甚至size为2的时候都轮不到执行这块代码
                if (objects.size() > maxPrimsInNode || minCost < leafCost)
                {
                    auto pmid = std::partition(objects.begin(), objects.end(), [=](Object* pi){
                            int b = nBuckets * centroidBounds.Offset(pi->getBounds().Centroid())[dim];
                            if (b == nBuckets) b = nBuckets - 1;
                            return b <= minCostSplitBucket;
                        });
                    auto beginning = objects.begin();
                    auto ending = objects.end();

                    auto leftshapes = std::vector<Object*>(beginning, pmid);
                    auto rightshapes = std::vector<Object*>(pmid, ending);

                    assert(objects.size() == (leftshapes.size() + rightshapes.size()));

                    node->left = recursiveBuild(leftshapes);
                    node->right = recursiveBuild(rightshapes);

                    node->bounds = Union(node->left->bounds, node->right->bounds);
                    break;
                }
                //如果这个划分效果还不如直接算，那就不break，直接执行下面的的
                //但是我的maxPrimsInNode设置的是1，所以无论如何都用sah，所以就先break下
                break;
            }
            case SplitMethod::NAIVE :
            {   switch (dim) {
            case 0:
                std::sort(objects.begin(), objects.end(), [](auto f1, auto f2) {
                    return f1->getBounds().Centroid().x <
                    f2->getBounds().Centroid().x;
                    });
                break;
            case 1:
                std::sort(objects.begin(), objects.end(), [](auto f1, auto f2) {
                    return f1->getBounds().Centroid().y <
                    f2->getBounds().Centroid().y;
                    });
                break;
            case 2:
                std::sort(objects.begin(), objects.end(), [](auto f1, auto f2) {
                    return f1->getBounds().Centroid().z <
                    f2->getBounds().Centroid().z;
                    });
                break;
            }

            auto beginning = objects.begin();
            auto middling = objects.begin() + (objects.size() / 2);
            auto ending = objects.end();

            auto leftshapes = std::vector<Object*>(beginning, middling);
            auto rightshapes = std::vector<Object*>(middling, ending);

            assert(objects.size() == (leftshapes.size() + rightshapes.size()));

            node->left = recursiveBuild(leftshapes);
            node->right = recursiveBuild(rightshapes);

            node->bounds = Union(node->left->bounds, node->right->bounds);
            break;
            }
        }


    }

    return node;
}
```

[![pCTuAL6.jpg](https://s1.ax1x.com/2023/07/18/pCTuAL6.jpg)](https://imgse.com/i/pCTuAL6)
