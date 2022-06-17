---
title: PAT-The Dominant Color & Set Similarity
date: 2022-06-16 23:57:00 +0800
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

## A1054. [The Dominant Color](https://pintia.cn/problem-sets/994805342720868352/problems/994805422639136768)

```cpp
#include<iostream>
#include<unordered_map>
using namespace std;

int main()
{
    int M,N;
    cin>>M>>N;
    unordered_map<string, int> m;//用来记录相同像素值的个数
    for(int i = 0; i < N; i++)
    {
        for(int j = 0; j < M; j++)
        {
            string color;
            cin>>color;
            m[color]++;
        }
    }
    int max = -1;
    string ans;
    //找到最大的
    for(auto each : m)
    {
        if(each.second > max)//如果某个个数大于max
        {
            ans = each.first;//更新主要颜色
            max = each.second;
        }
    }
    cout<<ans;
}
```

    直接使用哈希表轻松解决。

## A1063. [Set Similarity](https://pintia.cn/problem-sets/994805342720868352/problems/994805409175420928)

```cpp
#include<iostream>
#include<set>
#include<cstdio>
using namespace std;
int calShared(set<int> a, set<int> b)
{
    int cnt = 0;
     set<int>:: iterator  aIndex = a.begin(), bIndex = b.begin();//双指针
    //当两个指针都还指向合法的数据时
    while(aIndex != a.end() && bIndex != b.end())
    {
        //开始分情况
        if(*aIndex == *bIndex)//如果两个数字相等
        {
            cnt++;
            aIndex++;
            bIndex++;
        }
        else if(*aIndex > *bIndex)//如果a指针比b指针所指数字大
        {
            bIndex++;
        }
        else//如果b指针比a指针所指数字大
        {
            aIndex++;
        }
    }
    return cnt;
}
int main()
{
    int N,M,K;//N个set，每个set有M个数，有K次询问
    cin>>N;
    set<int> s[N + 1];
    for(int i = 1; i <= N; i++)//N个set
    {
        cin>>M;//一共有M个数存入一个set
        while(M--)
        {
            int num;
            cin>>num;
            s[i].insert(num);//自动去重加从小到大排序
        }
    }
    //处理询问
    cin>>K;
    while(K--)
    {
        int a, b;
        cin>>a>>b;//保存两个set的序号
        //计算两个set重叠的部分
        int shared = calShared(s[a], s[b]);
        //计算百分比并打印
        if(K != 0)//当不是打印最后一行时
        {
            printf("%.1f%%\n",(double)(100*shared) / (double)(s[a].size() + s[b].size() - shared));
        }
        else printf("%.1f%%",(double)(100*shared) / (double)(s[a].size() + s[b].size() - shared));
    }
}
```

    这道题乍一看挺简单，但还是得想那么一下。

    思路：将数据存放在set，会自动排序+去重，使用双指针法计算两个set重叠的部分，最后再计算比例

    注意：

* set\<int>即可，不需要两个int

* 初始化不能像vector那样用()，得用[]

* 不能像vector使用下标，得用迭代器

* 迭代器的声明：set\<int> :: iterator it = s.begin();//注意不要拼错

* 提取迭代器所指的数值：*it；

* printf打印%号时，要打两个%%

## 小结

    写完这道题时已经十一点多了，有点晚了，不写第三题了，回宿舍睡觉了。