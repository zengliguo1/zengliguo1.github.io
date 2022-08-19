---
title: PAT-LCA in a Binary Tree
date: 2022-08-17 21:47:00 +0800
categories: [算法刷题, PAT]
tags: [LCA, dfs]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1151 [LCA in a Binary Tree](https://pintia.cn/problem-sets/994805342720868352/problems/1038430130011897856)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

const int maxn = 10010;
vector<int> in(maxn);
vector<int> pre(maxn);
struct node
{
    int val;
    node* left = nullptr, * right = nullptr;
    node(int _val) : val(_val) {}
}*root;
int m, n;
node* build( int inL, int inR, int preL, int preR)
{
    if (inL > inR || preL > preR) return nullptr;
    node* root = new node(pre[preL]);
    int index;
    for (int i = inL; i <= inR; i++)
    {
        if (in[i] == pre[preL])
        {
            index = i;
            break;
        }
    }
    root->left = build(inL, index - 1,preL + 1, preL + index - inL);
    root->right = build(index + 1, inR, preL + index - inL + 1, preR);
    return root;
}
bool flag1, flag2;
int LCA = -1;
int search(node* root, int u, int v)
{
    if (root == nullptr) return 0;
    int cnt = 0;
    cnt = search(root->left, u, v) + search(root->right, u, v);
    if (root->val == u)
    {
        flag1 = true;
        cnt++;
    }
    if (root->val == v)
    {
        flag2 = true;
        cnt++;
    }
    if (LCA == -1 && cnt == 2)
    {
        LCA = root->val;
    }
    return cnt;
}
int main()
{
    //freopen("input.txt", "r", stdin);
    cin >> m >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> in[i];
    }
    for (int i = 0; i < n; i++)
    {
        cin >> pre[i];
    }
    root =  build(0, n - 1, 0, n - 1);
    for (int i = 0; i < m; i++)
    {
        int u, v;
        cin >> u >> v;
        LCA = -1;
        flag1 = false, flag2 = false;
        search(root, u, v);
        if (flag1 && flag2)
        {
            if (LCA == u)
            {
                printf("%d is an ancestor of %d.\n", u, v);
            }
            else if (LCA == v)
            {
                printf("%d is an ancestor of %d.\n", v, u);
            }
            else
            {
                printf("LCA of %d and %d is %d.\n", u, v, LCA);
            }
        }
        else if(!flag1 && !flag2)
        {
            printf("ERROR: %d and %d are not found.\n", u, v);
        }
        else if (flag1)
        {
            printf("ERROR: %d is not found.\n", v);
        }
        else
        {
            printf("ERROR: %d is not found.\n", u);
        }
    }
}
```

    这道题使用了中序+前序建树，和dfs搜索最邻近祖先，先搜索，返回左子树和右子树搜索到目标的数量的和，如果是2而且LCA仍然是初始值，那么就是最邻近祖先了，因为是深度优先搜索，从底往上搜。没能一次ac竟然是因为，`cin>>m>>n`写成了`cout<<m<<n`我说怎么上来就打印了俩0啊......第一次出这毛病，可能是白天和高中同学玩累了2333。
