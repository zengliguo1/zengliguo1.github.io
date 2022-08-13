---
title: PAT-Forwards on Weibo
date: 2022-08-12 17:38:00 +0800
categories: [算法刷题, PAT]
tags: [图的遍历]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1076 [Forwards on Weibo](https://pintia.cn/problem-sets/994805342720868352/problems/994805392092020736)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<cstdio>
using namespace std;
const int maxn = 1010;
vector<vector<int>> users(1010);
int n, l, k, forwards;//用户总人数、间接粉丝层数、询问个数、最大转发数
bool isVisited[maxn] = { false };
//回溯三部曲
//1.确认返回类型和参数
void dfs(int user, int level)
{
    //2.终止条件
    if (level > l) return;
    //3.每次回溯的遍历过程
    for (int fan : users[user])//遍历该user的粉丝以及粉丝的粉丝
    {
        if (!isVisited[fan])//如果没有遍历过
        {
            isVisited[fan] = true;//先置为true
            forwards++;
        }
        dfs(fan, level + 1);
    }
}
int main()
{
    cin >> n >> l;
    for (int i = 1; i <= n; i++)
    {
        int _num;
        scanf("%d", &_num);
        for (int j = 0; j < _num; j++)//遍历每个用户关注的人
        {
            int _follows;
            scanf("%d", &_follows);
            users[_follows].emplace_back(i);//给他关注的人的粉丝列表中添加自己
        }
    }
    cin >> k;
    for (int i = 0; i < k; i++)
    {
        int _user;
        cin >> _user;
        //重置最大转发数和visited数组
        forwards = 0;
        fill(isVisited + 1, isVisited + 1 + n, false);
        isVisited[_user] = true;//先把自己设置为true，也就是说自己不能算转发量
        dfs(_user, 1);
        //cout << forwards << endl;
        printf("%d\n", forwards);
    }
}
```

    这个使用dfs写的，但最后一组数据会超时（换成printf和scanf也超时），而且调试正确也花了段时间，这里面有一些坑：

* 遍历过的节点就不能继续dfs了吗？错误的，只有level超限了才不能dfs，因为如果最开始以level的层数（因为是深度优先遍历）遍历过一个节点（也就是说visted设置为true了），那么在回溯时，发现了这个节点，但level可能并没有超限，所以尽管遍历过它，仍然需要继续遍历它的邻接表，只是不用再多加一次转发数了

* 由于dfs最后一组会超时，所以还是采用bfs吧

```cpp
#include<iostream>
#include<vector>
#include<queue>
using namespace std;

const int maxn = 1010;
vector<vector<int>> fans(maxn);//每个用户的粉丝列表
vector<int> layer(maxn);//记录每个用户的层数
bool isVisited[maxn] = { false };
int n, l, k;//用户总数、层数、询问次数
//广度优先遍历
int BFS(int user)
{
    queue<int> q;//储存粉丝的队列
    q.push(user);//先把自己加进去
    //设置自己为0层
    layer[user] = 0;
    fill(isVisited + 1, isVisited + 1 + n, false);//重置visited数组
    isVisited[user] = true;//先把自己设置为true
    int forwards = 0;//统计转发数
    while (!q.empty())
    {
        int top = q.front();
        q.pop();
        for (int fan : fans[top])//遍历自己的每一个粉丝
        {
            if (isVisited[fan]) continue;//如果之前遍历过，直接跳过
            layer[fan] = layer[top] + 1;//自己粉丝的层数为自己的+1
            if (layer[fan] <= l)//如果没遍历过，而且所在层数小于等于l
            {
                isVisited[fan] = true;
                forwards++;//转发数+1
                q.push(fan);//将这个粉丝加入队列
            }
        }
    }
    return forwards;
}
int main()
{
    cin >> n >> l;
    for (int i = 1; i <= n; i++)
    {
        int _num;//关注人数
        cin >> _num;
        for (int j = 0; j < _num; j++)
        {
            int _follows;
            cin >> _follows;
            fans[_follows].emplace_back(i);//将自己加入到关注的人的粉丝列表
        }
    }
    cin >> k;
    for (int i = 0; i < k; i++)
    {
        int _user;
        cin >> _user;
        cout << BFS(_user) << endl;
    }
}
```

    这个BFS还是有点坑的：

* 关于层数的增加时机，不能在遍历到粉丝的开始就将当前粉丝置为top的层数+1，这样会导致一种情况：更改某个用户所在的层数。什么意思呢？就是说，其实每个用户在整个遍历过程中有且只有一个对应的层数。假设用户A有粉丝B和C，C有粉丝B，一旦在用户C的粉丝列表那里再一次遇到已经遍历过的用户B，如果上来就让B的层数+1，显然就更改了B的层数，那么在遍历B的粉丝的时候，B的粉丝层数就会多1，这样就会导致最终的转发数减少。所以，遇到遍历过的节点，直接continue跳过最方便了。
