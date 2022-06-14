---
title: PAT-Raffle for Weibo Followers & Shortest Distance & A+B and C (64bit)
date: 2022-06-14 18:44:00 +0800
categories: [算法刷题, PAT]
tags: [模拟]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1124. [Raffle for Weibo Followers](https://pintia.cn/problem-sets/994805342720868352/problems/994805350803292160)

```cpp
#include<iostream>
#include<unordered_map>
using namespace std;
int main()
{
    int M,N,S;//M:粉丝人数，N:中奖人数间隔, S:开始的人序号（从1开始）
    cin>>M>>N>>S;
    if(S > M) cout<<"Keep going...";//开始序号大于粉丝数
    else
    {
        unordered_map<string, int> m;//用来查重
        string names[M + 1];
        //将名字放在数组中
        for(int i = 1; i <= M; i++)cin>>names[i];
        //开始抽奖
        int index = S;
        m[names[S]]++;//加入map
        cout<<names[S];
        index += N;
        while(index <= M)
        {
            if(m.count(names[index]))//如果已经中过奖
            {
                index++;
            }
            else//如果没中过奖
            {
                m[names[index]]++;//加入map
                cout<<endl<<names[index];
                index += N;
            }
        }

    }
}
```

    这道题太折磨了，本来挺简单的题，让我玩复杂了。我一开始为了省这一千个数组的空间复杂度，想挨个遍历，结果就问题重重，用取余来算，还有特判，最后只拿了17分（满分20），最后一个样例没通过，后来又改，结果改的四不像。最后还是选择了最简单的思路拿最多的分。这道题给我的教训是，拿分就行了，别老搁那想优化了嗷，优化半天代码量蹭蹭往上涨，bug也蹭蹭涨，就省那么点空间复杂度，真不值。

    PAT全英文+大量description，多少有点折磨了，但还是得适应，想念leetcode的第2天。

## A1046. [Shortest Distance](https://pintia.cn/problem-sets/994805342720868352/problems/994805435700199424)

```cpp
#include<iostream>
#include<cmath>
#include<vector>
using namespace std;

int main()
{
    //数据读入
    int N, M;//N:节点个数, M:询问次数
    cin>>N;
    //将每个节点的举例储存在数组中
    vector<int> dis(N + 1);//因为要从1开始，所以多申请一个空间
    for(int i = 1; i <= N; i++)
    {
        cin>>dis[i];
    }
    //储存前缀和
    vector<int> pre(N + 1);
    int sum = 0;
    pre[0] = dis[N];//0节点就储存从最后一个节点走到开头的距离
    //计算前缀和，计算从1节点到该位置需要走的距离
    for(int i = 1; i <= N; i++)
    {
        pre[i] = sum;
        sum += dis[i];
    }
    //开始处理询问
    cin>>M;
    while(M--)
    {
        int x,y;//储存两个位置
        cin>>x>>y;//读入
        //计算最短距离，两种情况，要么直达，要么反向直达
        int minIndex = min(x, y);//获取小坐标
        int maxIndex = max(x, y);//获取大坐标
        if(M != 0)//没处理到最后一个，都输出换行
        {
            //pre[maxIndex] - pre[minIndex] = 正向直达，从小坐标直接走到大坐标
            //pre[N] - pre[maxIndex] + pre[minIndex] + pre[0] = 反向直达
            //先从大坐标走到结尾，再从结尾走到小坐标
            //二者取小值打印
            cout<<min(pre[maxIndex] - pre[minIndex], pre[N] - pre[maxIndex] + pre[minIndex] + pre[0]);
            cout<<endl;
        }
        else//处理到最后一个, 不输出换行
        {
            cout<<min(pre[maxIndex] - pre[minIndex], pre[N] - pre[maxIndex] + pre[minIndex] + pre[0]);
        }
    }
}
```

    这道题是典型的前缀和，没什么多说的，需要注意的就是前缀和算哪一部分，包不包含自己节点，还有就是边界处理。

    由于PAT的题需要写头文件，所以经常会因为漏写头文件而导致编译错误，还要继续适应。

## A1065. [A+B and C (64bit)](https://pintia.cn/problem-sets/994805342720868352/problems/994805406352654336)

```cpp
#include<iostream>
#include<cstdio>
using namespace std;
int main()
{
    int N;//记录有几组数据
    scanf("%d", &N);
    for(int i = 1; i <= N; i++)
    {
        long long a,b,c;
        bool flag;
        scanf("%lld %lld %lld", &a, &b, &c);
        long long sum = a + b;//求和
        //判断溢出情况：
        //正溢出，肯定大于c
        if(a > 0 && b > 0 && sum <= 0)flag = true;
        //负溢出，肯定小于c
        else if(a < 0 && b < 0 && sum >= 0)flag = false;
        //正常情况
        else flag = sum > c;
        if(flag)//
        {
            //cout<<"Case #"<<i<<": "<<"true"<<endl;
            printf("Case #%d: true\n", i);
        }
        else printf("Case #%d: false\n", i);
            //cout<<"Case #"<<i<<": "<<"false"<<endl;
    }
}
```

    这道题，我起初想用字符串的运算，但好麻烦，看了别人的，直接算，去判断是否溢出就行了。而且有一件十分诡异的事情，就是，同样的代码，使用cin和cout的话，最后一个测试点就无法通过，但使用printf和scanf的话，就能通过，而且错误不是超时，是答案错误。

    这道题再次给我的教训就是，能暴力模拟就别想着花里胡哨。
