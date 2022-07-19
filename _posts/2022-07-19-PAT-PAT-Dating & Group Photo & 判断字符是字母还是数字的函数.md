---
title: PAT-Dating & Group Photo & 判断字符是字母还是数字的函数
date: 2022-07-19 20:40:00 +0800
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

## A1061 [Dating](https://pintia.cn/problem-sets/994805342720868352/problems/994805411985604608)

```cpp
#include<iostream>
#include<cstdio>
#include<vector>
#include<string>
using namespace std;
string Day[7] = {"MON", "TUE", "WED", "THU", "FRI", "SAT", "SUN"};
int main()
{
    //读取四个字符串
    string s1,s2,s3,s4;
    cin>>s1>>s2>>s3>>s4;
    //循环次数取两个字符串中短的那个
    int loop1 = s1.size() < s2.size() ? s1.size() : s2.size();
    int loop2 = s3.size() < s4.size() ? s3.size() : s4.size();
    //开始比较：
    int flag = 1;//1代表还在查DAY，2代表正在查HOUR
    for(int i = 0; i < loop1; i++)
    {
        //如果在对比的过程中发现了相同的字符
        if(s1[i] == s2[i])
        {
            //要打印Day
            if(flag == 1 && s1[i] >= 'A' && s1[i] <= 'G')
            {
                cout<<Day[s1[i] - 'A']<<' ';
                flag++;
            }
            else if(flag == 2)//打印HOUR
            {
                if(s1[i] >= '0' && s1[i] <= '9')
                {
                    printf("%02d:", s1[i] - '0');
                    break;
                }
                else if(s1[i] >= 'A' && s1[i] <= 'N')
                {
                    cout<<s1[i] - 'A' + 10<<':';
                    break;
                }
            }
        }
    }

    //打印MINUTE
    for(int j = 0; j < loop2; j++)
    {
        if(s3[j] == s4[j] && isalpha(s3[j]))
        {
            printf("%02d", j);
        }
    }
}
```

    这道题有两个难点：

* 首先就是题意，第一遍读题完全没看懂解码的原理，因为漏看了common这个单词，我把它译为了普通，但应该是相同，这样解释就合理了，当两个字符相同时，根据字符的情况来打印时间

* 第二个其实就是细节，要注意打印时间时都是打印两位，后面的分钟打印两位不容易忘记，是因为测试样例就是04，小时如果是10以内的话也要 打印0，不能直接打印字符，不然会有几个点过不去

## 判断字符是字母还是数字的函数

字母（不区分大小写）：isalpha();

大写字母：isupper();

小写字母：islower();

数字：isdigit();

字母和数字：isalnum();

转化为大写：toupper();

转化为小写：tolower();

## A1109 [Group Photo](https://pintia.cn/problem-sets/994805342720868352/problems/994805360043343872)

```cpp
#include<iostream>
#include<utility>
#include<vector>
#include<algorithm>
using namespace std;

bool comp(pair<string, int>& p1, pair<string, int>& p2)
{
    if (p1.second == p2.second)//如果身高相等，名字小的在前面
    {
        return p1.first < p2.first;
    }
    else//身高不一样的话，高的排前面
    {
        return p1.second > p2.second;
    }
}
int main()
{
    int N, K;//N个人，K行
    vector< pair<string, int>> people;//每个人的姓名、身高信息
    //读入数据
    cin >> N >> K;
    int remainder = N % K;//人数除以行数的余数，代表最后一行要多站的人
    int rowNum = N / K;//除了最后一行，每行要站的人数
    int nearNum = remainder + rowNum;//最后一行站的人
    while (N--)
    {
        string name;
        int height;
        cin >> name >> height;
        people.emplace_back(make_pair(name, height));
    }
    //按从高到低排序（身高一样的话，按名字升序）
    sort(people.begin(), people.end(), comp);
    //开始排队，从最后一排开始排
    vector< pair<string, int>> everyQueue;
    //先把最高的放进队列
    everyQueue.emplace_back(people[0].first, people[0].second);
    for (int i = 1; i < nearNum; i++)
    {
        //单数插在队列左边
        if (i % 2 == 1)
        {
            everyQueue.insert(everyQueue.begin(), people[i]);
        }
        else//双数插在队列右边
        {
            everyQueue.emplace_back(people[i]);
        }
    }
    //打印最后一行的人名
    for (int i = 0; i < everyQueue.size(); i++)
    {
        if (i == 0)cout << everyQueue[i].first;
        else cout << ' ' << everyQueue[i].first;
    }
    cout << endl;
    //处理后面的几行
    for (int i = 0; i < K - 1; i++)//遍历每一行
    {
        //先清空队列
        everyQueue.clear();
        //第一个人插入队列（中间）
        everyQueue.emplace_back(people[nearNum + i * rowNum]);
        for (int j = 1; j < rowNum; j++)//后面的人按规则入队
        {
            if (j % 2 == 1)//单数站左边
            {
                everyQueue.insert(everyQueue.begin(), people[nearNum + i * rowNum + j]);
            }
            else//双数站右边
            {
                everyQueue.emplace_back(people[nearNum + i * rowNum + j]);
            }
        }
        //打印这一行的人名
        for (int i = 0; i < everyQueue.size(); i++)
        {
            if (i == 0)cout << everyQueue[i].first;
            else cout << ' ' << everyQueue[i].first;
        }
        //不是第一行人就打印换行
        if (i != K - 2) cout << endl;
    }


}
```

    这道题有两个出错的地方

* 第一个是报错，数组下标越界Expression:vector subscript out of range。这是为什么呢？这是因为我最初算remainder、rowNum、nearNum的时候，是在读取完人名和身高后算的，这个时候N已经减到0了，所以算出的三个变量是不对的，0%K算的是-1，所以越界了。

* 第二个是排的顺序，相同身高下是名字小的先去站队，我给搞反了
  
  
