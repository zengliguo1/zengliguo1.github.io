---
title: PAT-Boys vs Girls & Shuffling Machine & Look-and-say Sequence
date: 2022-06-13 13:42:00 +0800
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

## 1036.[Boys vs Girls](https://pintia.cn/problem-sets/994805342720868352/problems/994805453203030016)

```cpp
#include<iostream>
#include<algorithm>
using namespace std;
struct stu
{
    string name;
    char gender;
    string ID;
    int grade;
};
bool compM(stu s1, stu s2)
{
    return s1.grade < s2.grade;
}
bool compF(stu s1, stu s2)
{
    return s1.grade > s2.grade;
}
int main()
{
    int N;
    cin>>N;
    stu f[N];//女学生
    stu m[N];//男学生
    int mNum = 0, fNum = 0;//分别代表男女生数量
    //数据输入
    while(N--)
    {
        string _name, _ID;
        char _gender;
        int _score;
        cin>>_name>>_gender>>_ID>>_score;
        if(_gender == 'M')//男学生
        {
            m[mNum].name = _name;
            m[mNum].ID = _ID;
            m[mNum].grade = _score;
            mNum++;
        }
        else//女学生
        {
            f[fNum].name = _name;
            f[fNum].ID = _ID;
            f[fNum].grade = _score;
            fNum++;
        }
    }
    //排序
    if(fNum > 0)
    {
        sort(f, f + fNum, compF);
        cout<<f[0].name<<' '<<f[0].ID<<endl;
    }
    else cout<<"Absent"<<endl;
    if(mNum > 0)
    {
        sort(m, m + mNum, compM);
        cout<<m[0].name<<' '<<m[0].ID<<endl;
    }
    else cout<<"Absent"<<endl;
    if(mNum == 0 || fNum == 0)
    {
        cout<<"NA";
    }else cout<<f[0].grade - m[0].grade;


}
```

    PAT的题目跟力扣一个很大的不同就是需要自己写输入，而且一旦自己写输入就意味着代码量比力扣的多很多，这道题目并不难，就是变量太多了，测试的时候有很多编译错误就是变量打错了......

## 1042. [Shuffling Machine](https://pintia.cn/problem-sets/994805342720868352/problems/994805442671132672)

```cpp
#include<iostream>
#include<vector>
#include<string>
using namespace std;
int main()
{
    int N;//洗牌次数
    vector<int> pos(55);//洗牌位置
    vector<string> cards;//牌,不要声明数组大小！不然用不了emplace_back
    vector<string> tmp(55);//临时数组
    //数据输入,牌先初始化（题目竟然不给初始化，岂可修！）
    cards.emplace_back("Null");//先把0位置占掉
    for(int i = 0; i < 4; i++)
    {
        string s1;
        switch(i)
        {
            case 0 : s1 = "S";break;
            case 1 : s1 = "H";break;
            case 2 : s1 = "C";break;
            case 3 : s1 = "D";break;
        }
        for(int j = 1; j <= 13; j++)
        {
            cards.emplace_back(s1 + to_string(j));
        }
    }
    cards.emplace_back("J1");
    cards.emplace_back("J2");
    cin>>N;
    for(int i = 1; i <= 54; i++)cin>>pos[i];
    //开始洗牌
    while(N--)
    {
        //先把牌放到临时数组中
        for(int i = 1; i <= 54; i++)
        {
            //把当前i位置的牌放到pos[i]的位置
            tmp[pos[i]] = cards[i];
        }
        //再把临时数组的牌换成cards
        for(int i = 1; i <= 54; i++)
        {
            cards[i] = tmp[i];
        }
    }
    for(int i = 1; i <= 54; i++)
    {
        cout<<cards[i];
        if(i != 54) cout <<' ';
    }
}
```

    emmm，这道题最需要注意的地方是，如果初始化给vector了固定空间，那么就不能用emplace_back()函数了！！我这里没有把牌做到0~53的映射，直接用string存了，写了50行代码，我看有个博主写的映射，写了30行代码，但我觉得我这个也挺好读的哈哈哈。

## 1140. [Look-and-say Sequence](https://pintia.cn/problem-sets/994805342720868352/problems/994805344490864640)

```cpp
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;

int main()
{
    int D, N;
    cin>>D>>N;
    string d = to_string(D);
    N -= 1;//第N项，其实是算了N - 1次    
    while(N--)
    {
        //左右双指针
        int l = 0, r = 1;
        string tmp;//临时string，用来存n + 1项
        for(; r < d.size(); r++)//遍历右指针
        {
            if(d[r] != d[l])//如果右指针和左指针指的数不一样了
            {
                tmp += d[l] + to_string(r - l);//左指针所指数字+次数
                l = r;
            }
        }
        //把最后的数字也加入tmp,无论最后是都相等还是最后一个不相等了，都适用
        tmp += d[l] + to_string(r - l);
        d = tmp;//更新d
    }
    cout<<d;

}
```

    首先，这道题要先弄清楚题意，不是统计上一个字符串中每种数字一共有几个，而是连续的有几个，一旦不连续了就要重新算的。比如111222333111222就是：1323331323（3个1，3个2，3个3，3个1，3个2）而不是162633.

    第二，想用memset就要包含`<cstdio>`和`<cstring>`这两个头文件才行

## 小结

    今天开始刷pat，比想象中的更费时间，主要还是得熟悉熟悉这个答题的格式，毕竟跟leetCode还是有点区别。每天尽量三道题吧，这样一个暑假应该是能把所有类型刷一遍。
