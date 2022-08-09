---
title: PAT-Highest Price in Supply Chain & The Largest Generation & Lowest Price in Supply Chain & Path of Equal Weight & Complete Binary Search Tree
date: 2022-08-09 17:22:00 +0800
categories: [算法刷题, PAT]
tags: [树的遍历, BST]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1090 [Highest Price in Supply Chain](https://pintia.cn/problem-sets/994805342720868352/problems/994805376476626944)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
using namespace std;
//节点结构体
struct node
{
    vector<int> child;
};
int n;//节点总数
double price, r;//单价和涨幅
double highest = 0.0;//最高价格
int retailersNum = 0;//销售最高价的零售商个数
const int maxn = 100010;
int root;//根节点下标
vector<node> supply(maxn);
//回溯三部曲
//1.确认返回类型和参数
void travelsal(int root, int depth)
{
    //2.终止条件
    //先判断是不是叶子节点
    if (supply[root].child.empty())//是叶子节点，算钱,中止
    {
        double tmp = (price * pow(1 + r, depth));
        if (tmp > highest)
        {
            highest = tmp;//更新最大值
            retailersNum = 1;//刷新人数
        }
        else if (tmp == highest)
        {
            retailersNum++;
        }
    }
    else//不是叶子节点
    {
        //3.每个回溯的遍历过程
        for (int c : supply[root].child)
        {
            travelsal(c, depth + 1);
        }
    }
}
int main()
{
    cin >> n >> price >> r;
    r /= 100;
    for (int i = 0; i < n; i++)
    {
        int _num;
        cin >> _num;//自己的供应商
        if (_num == -1)//说明是根节点
        {
            root = i;
        }
        else//说明_num是i的供应商
        {
            supply[_num].child.emplace_back(i);
        }
    }
    travelsal(root, 0);
    printf("%.2f %d", highest, retailersNum);
}
```

    这道题和昨天做的最后一题几乎一样，就是要算的东西不一样，尽管如此也有需要注意的地方。

* 算价钱时要用`double`不能用`int`

* 开始中断显示数组越界，我就猜哪里可能出-1了，果然，在读取数据时，遇到根节点`root=i`才对，不能是`root=_num`，因为此时`_num`是-1。

## A1094 [The Largest Generation](https://pintia.cn/problem-sets/994805342720868352/problems/994805372601090048)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
using namespace std;
//节点结构体
struct node
{
    vector<int> children;
};
int n, m;//家族总人数，有孩子的人数
vector<node> family(100);//家族静态树
vector<int> population(100, 0);//记录每层的人口
int maxPop = 0;
int maxDep = 0;
//回溯三部曲
//1.确认返回类型，参数
void travelsal(int root, int depth)
{
    //先计算本层孩子
    population[depth]++;
    if (population[depth] > maxPop)//更新人口最大代
    {
        maxPop = population[depth];
        maxDep = depth;
    }
    //2.中止条件
    if (family[root].children.empty()) return;
    else
    {
        //3.每次回溯的遍历
        for (int c : family[root].children)
        {
            travelsal(c, depth + 1);
        }
    }
}
int main()
{
    cin >> n >> m;
    //读取家族树
    for (int i = 0; i < m; i++)
    {
        int _index, _num;
        cin >> _index >> _num;
        for (int j = 0; j < _num; j++)
        {
            int _child;
            cin >> _child;
            family[_index].children.emplace_back(_child);
        }
    }
    travelsal(1, 1);
    cout << maxPop << ' ' << maxDep;
}
```

    依旧是遍历，并记录深度，更之前的两道题很类似。

## A1106 [Lowest Price in Supply Chain](https://pintia.cn/problem-sets/994805342720868352/problems/994805362341822464)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
using namespace std;
//节点结构体
struct node
{
    vector<int> children;
};
int n;
double price, r;
int maxn = 100010;
vector<node> supply(maxn);
double lowest = 1e10;//最低价
int lowNum = 0;//最低价人数


//回溯三部曲
//1.确认返回类型和参数
void travelsal(int root, int depth)
{
    //2.中止条件
    if (supply[root].children.empty())//是叶子节点
    {
        double tmp = price * pow(1 + r, depth);
        if (tmp < lowest)//更新最低价格
        {
            lowest = tmp;
            lowNum = 1;
        }
        else if (tmp == lowest)
        {
            lowNum++;
        }
    }
    else//3.每次回溯的遍历
    {
        for (int c : supply[root].children)
        {
            travelsal(c, depth + 1);
        }
    }
}

int main()
{
    cin >> n >> price >> r;
    r /= 100;
    for (int i = 0; i < n; i++)
    {
        int _num;
        cin >> _num;
        if (_num != 0)//说明有孩子
        {
            for (int j = 0; j < _num; j++)
            {
                int _child;
                cin >> _child;
                supply[i].children.emplace_back(_child);
            }
        }
    }
    travelsal(0, 0);
    printf("%.4f %d", lowest, lowNum);
}
```

    还是供应商问题，出现了段错误，查了下，发现是`maxn`初始化小了，初始化成一万了，应该是十万，不然有的数据会越界。

## A1053 [Path of Equal Weight](https://pintia.cn/problem-sets/994805342720868352/problems/994805424153280512)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
#include<algorithm>
using namespace std;
//节点结构体
struct node
{
    int weight;
    vector<int> children;
};
const int maxn = 100;
int n, m, s;//节点总数、非叶子节点数、目标权重和
vector<node> tree(maxn);
vector< vector<int>> path;
vector<int> tmpPath;
//回溯三部曲
//1.确认返回类型和参数
void travelsal(int root, int sum)
{
    //2.中止条件:和已经超过s，或者满足条件
    if (sum > s) return;
    if (tree[root].children.empty() && sum == s)
    {
        //先把临时路径存到总路径中
        path.emplace_back(tmpPath);
        return;//也返回，不继续找了，因为都是正数
    }

    //3.每次回溯的遍历
    for (int c : tree[root].children)
    {
        tmpPath.emplace_back(tree[c].weight);//先把c的权重加入临时路径
        travelsal(c, sum + tree[c].weight);
        tmpPath.pop_back();//把c再弹出来
    }
}

bool comp2(vector<int>& p1, vector<int>& p2)
{
    int times = min(p1.size(), p2.size());
    for (int i = 0; i < times; i++)
    {
        if (p1[i] != p2[i])
        {
            //大的在前面
            return p1[i] > p2[i];
        }
    }
    //完全相同的
    return false;
}
int main()
{
    cin >> n >> m >> s;
    //读取每个节点的权重
    for (int i = 0; i < n; i++)
    {
        cin >> tree[i].weight;
    }
    //读取每个非叶子节点的孩子
    for (int i = 0; i < m; i++)
    {
        int _id, _num;
        cin >> _id >> _num;
        for (int j = 0; j < _num; j++)
        {
            int _child;
            cin >> _child;
            tree[_id].children.emplace_back(_child);
        }
    }
    tmpPath.insert(tmpPath.begin(), tree[0].weight);
    travelsal(0, tree[0].weight);

    //找完所有的路径了，开始排序
    //把所有的路排一下
    sort(path.begin(), path.end(), comp2);
    //输出
    for (auto p : path)
    {
        for (int i = 0; i < p.size(); i++)
        {
            if (i != 0) cout << ' ' << p[i];
            else cout << p[i];
        }
        cout << endl;
    }
}
```

    这道题看似不难，但实际上坑很多，题目比较绕。

* 首先需要注意的是，放入路径的不是节点的序号，而是节点的权重

* 其次，题目中所说的**non-increasing**指的是所有的路径之间是非升序的，而不是单个路径中的权重是非升序的，也就是说不需要对单个路径的权重单独排序

* 最后有一处中断，就是一直提示是比较器错误，那么也就是sort函数的comp函数的问题，debug后发现，如果两个序列完全一样的话，return true就会比较错误，return false就不会比较错误了，怪。还好测试样例有两个一样的数据，不然如果是别的样例，那我又不知道要查多久了

## A1064 [Complete Binary Search Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805407749357568)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
#include<algorithm>
using namespace std;
const int maxn = 1010;
vector<int> CBT(maxn);//静态二叉树
vector<int> val(maxn);
int n;//节点个数
int index = 0;//val数组下标
//中序遍历
void in(int root)
{
    //中止条件
    if (root > n) return;
    //完全bst也就是cbt的节点，左孩子下标为root*2，有孩子下标为root*2 + 1
    in(root * 2);
    CBT[root] = val[index++];
    in(root * 2 + 1);
}
int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> val[i];
    }
    //给节点的值排序
    sort(val.begin(), val.begin() + n);
    //bst的中序遍历是有序的
    in(1);
    //打印结果，按顺序就是层序遍历的结果
    for (int i = 1; i <= n; i++)
    {
        if (i == 1) cout << CBT[i];
        else cout << ' ' << CBT[i];
    }

}
```

    二叉搜索树，想做这道题，需要了解两个知识点：

* 二叉搜索树的中序遍历是递增的

* 完全二叉搜索树（静态树）的某个节点下标为root，其左孩子下标就是root×2，其右孩子下标是root×2 + 1
