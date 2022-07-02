---
title: PAT-Speech Patterns & Read Number in Chinese & string的erase()用法
date: 2022-07-02 14:25:00 +0800
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

## A1071. [Speech Patterns](https://pintia.cn/problem-sets/994805342720868352/problems/994805398257647616)

```cpp
#include<iostream>
#include<unordered_map>
#include<string>
#include<cctype>
using namespace std;
int main()
{
    string s;//用来存储input
    getline(cin, s);
    unordered_map<string, int> map;//用来记录单词个数
    //遍历s
    string word;
    for(int i = 0; i < s.size(); i++)
    {
        word.clear();//先清空word
        //如果没越界而且当前字符是字母或者数字
        while(i < s.size() && isalnum(s[i]))
        {
            word += tolower(s[i]);//cctype的函数
            i++;
        }
        //如果不是空string就加1
        if(!word.empty()) map[word]++;
    }
    //查找次数最多的单词
    int maxC = 0;//记录最大次数
    string maxS;//记录对应的字符串
    for(auto it = map.begin(); it != map.end(); it++)
    {
        if(it -> second > maxC)//如果大于最大值，更新
        {
            maxC = it -> second;
            maxS = it -> first;
        }
    }
    cout<<maxS<<' '<<maxC;

}
```

    这道题就是去找句子中出现次数最多的单词是什么，有几个。

    先获取整个句子，再挨个遍历字符，其中用到了两个函数，分别是isalnum()和tolower()。

* tolower()语法

>  #include <cctype>
> 
>   int tolower( int ch );
> 
>   int toupper( int ch );
> 
> 可以将字符ch转换为小写、大写

* isalnum()语法

>  #include <cctype>
> 
>   int isalnum( int ch );
> 
>   如果参数是数字或字母字符，函数返回非零值，否则返回零值。
