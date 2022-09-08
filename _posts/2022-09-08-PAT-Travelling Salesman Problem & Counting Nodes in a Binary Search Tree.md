---
title: PAT-Travelling Salesman Problem & Counting Nodes in a Binary Search Tree
date: 2022-09-08 12:30:00 +0800
categories: [算法刷题, PAT]
tags: [图, 哈希表, BST]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    承接昨天的题，做完这两道也就花了一个多小时，最后还剩1小时1分钟，而且四道题都ac了，这套题还是很简单的，明天把哈夫曼树看看，做几道哈夫曼树的题目吧。

    得益于昨天背了背c++的基础知识点，全局变量、数组是在**编译阶段**申请在**全局/静态存储区**的，如果不初始化的话，会默认赋值成0，数组的全部元素也是0，bool的话是false。

    得益于前两看了看柳神的代码，于是我也开始精简代码行数，发现确实能节省不少时间。

## A1150 [Travelling Salesman Problem](https://pintia.cn/problem-sets/994805342720868352/problems/1038430013544464384)

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
#include<cstdio>
using namespace std;
const int maxn = 210, inf = 0x3fffffff;
int N, M, k, n, neibghbor[maxn][maxn], u, v, w, mindis = inf, minpath;

int main()
{
    cin >> N >> M;
    fill(neibghbor[0], neibghbor[0] + maxn * maxn, inf);
    for (int i = 0; i < M; i++)
    {
        cin >> u >> v >> w;
        neibghbor[u][v] = w;
        neibghbor[v][u] = w;
    }
    cin >> k;
    for (int i = 1; i <= k; i++)
    {
        cin >> n;
        vector<int> path(n);
        unordered_map<int, int> map;
        int tmpdis = 0;
        bool flag = false;
        for (int j = 0; j < n; j++)
        {
            cin >> path[j];
            map[path[j]]++;
            if (j > 0)
            {
                if (neibghbor[path[j]][path[j - 1]] == inf)
                {
                    flag = true;
                }
                else tmpdis += neibghbor[path[j]][path[j - 1]];
            }
        }
        if (flag) printf("Path %d: NA (Not a TS cycle)\n", i);
        else
        {
            if (path.front() == path.back() && map.size() == N)
            {
                if(path.size() == N + 1) printf("Path %d: %d (TS simple cycle)\n", i, tmpdis);
                else printf("Path %d: %d (TS cycle)\n", i, tmpdis);
                if (tmpdis < mindis)
                {
                    mindis = tmpdis;
                    minpath = i;
                }
            }
            else printf("Path %d: %d (Not a TS cycle)\n", i, tmpdis);
        }
    }
    printf("Shortest Dist(%d) = %d\n", minpath, mindis);
}
```

    这道题目很简单，但是第一次提交只对了8分，前两个测试点没过，查了下，发现了老错误，我再判断中间有路不连通的时候，直接就break了，但后面很有可能还有节点没有读取，那么就会导致出错，要么就不break，只是改下flag，要么就用getline把剩下的全吸收了。

## A1115 [Counting Nodes in a Binary Search Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805355987451904)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;
struct node
{
    int v;
    node* left = nullptr, * right = nullptr;
    node(int _v) : v(_v) {}
}*Root = nullptr;
const int maxn = 10010;
int n, lowestl = 0, nodesN[maxn], tmp;
void Insert(node*& root, int val)
{
    if (root == nullptr) root = new node(val);
    else if (val <= root->v) Insert(root->right, val);
    else Insert(root->left, val);
}
void travelsal(node* root, int level)
{
    if (root == nullptr) return;
    nodesN[level]++;
    if (level > lowestl) lowestl = level;
    travelsal(root->left, level + 1);
    travelsal(root->right, level + 1);
}
int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> tmp;
        Insert(Root, tmp);
    }
    travelsal(Root, 0);
    printf("%d + %d = %d", nodesN[lowestl], nodesN[lowestl - 1], nodesN[lowestl] + nodesN[lowestl - 1]);
}
```

    亏这道题还30分呢，这么简单，就一建树和遍历，但就这我还debug了会，有两点：

* 我的insert函数，在当值小于当前节点值时，应当插入左子树，我给写成`right`了，怪不得一开始是1+1=2的结果

* 注意insert的形参root，要加引用，不然不会真的把值加在树上的
