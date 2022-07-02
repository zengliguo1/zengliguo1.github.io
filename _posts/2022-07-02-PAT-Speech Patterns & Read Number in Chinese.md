---
title: PAT-Speech Patterns & Read Number in Chinese 
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

## A1082. [Read Number in Chinese](https://pintia.cn/problem-sets/994805342720868352/problems/994805385053978624)

```cpp
#include<iostream>
#include<string>
#include<vector>
using namespace std;

int main()
{
    string num;
    cin>>num;//读取input
    vector<string> ans;//记录最终答案
    vector<string> cnNum = {"ling", "yi", "er", "san", "si", "wu", "liu", "qi", "ba", "jiu"};//存储中文数字
    vector<string> smallUnit = {"Shi", "Bai", "Qian"};//存储中文单位，万和亿单独算
    vector<string> biglUnit = {"Wan", "Yi"};
    //如果是负数
    if(num[0] == '-')
    {
        ans.emplace_back("Fu");
        num.erase(0, 1);//删除字符串的第0位
    }
    //处理前导0
    while(num.size() > 0 && num[0] == '0')
    {
        num.erase(0, 1);//如果前面有0，则删除掉，直到删完或者不再是前导0
    }
    //全是0的情况
    if(num.size() == 0)
    {
        cout<<"ling";
        return 0;
    }
    int len = num.size();
    //每四个一组来输出,不用担心当len = 8或4的时候，因为这时候，后面的循环是不会执行的，相当于跳过了
    for(int group = len / 4; group >= 0; group--)
    {
        //两个标记，用来标记前面是否出现过0，和是否输出过数字
        bool hasZero = false, hasPrint = false;
        //j从每一组的第一个数字开始len - group * 4 - 4其实是算下一个组的起始下标
        for(int j = max(0, len - group * 4 - 4), k = len - group * 4; j < k; j++)
        {
            if(num[j] - '0' == 0)//如果有0
            {
                hasZero = true;
            }
            else//当这一位不是0时
            {
                hasPrint = true;
                if(hasZero)//前面有0
                {
                    ans.emplace_back("ling");
                    hasZero = false;
                }
                ans.emplace_back(cnNum[num[j] - '0']);//插入当前位的数字
                int dis = k - j;//通过j下标到本组最远位置k的距离来判断输出什么单位
                if(dis >= 2)//距离等于2时，是十
                {
                    ans.emplace_back(smallUnit[dis - 2]);
                }
            }
        }
        //如果这一组大于0，而且打印过数字的话
        if(group > 0 && hasPrint)
        {
            ans.emplace_back(biglUnit[group - 1]);
        }
    }
    //打印结果
    for(int i = 0; i < ans.size(); i++)
    {
        if(i == 0)cout<<ans[i];
        else cout<<' '<<ans[i];
    }
}
```

    这道题目究极复杂，自己想半天想不明白，借鉴了别人的方法，要先处理符号和前导0，最后根据标记来插入元素。空格可以最后输出的时候考虑，一开始我想直接就打印了，发现空格确实是个比较难想的东西，但如果放在最后处理的话就很简单了。

    字符串的erase函数用法：

* 语法

>   [iterator](iterators.html) erase( [iterator](iterators.html) pos );
>   [iterator](iterators.html) erase( [iterator](iterators.html) start, [iterator](iterators.html) end );
>   basic_string &erase( size_type index = 0, size_type num = npos );
> 
> erase()函数可以:
> 
> - 删除pos指向的字符, 返回指向下一个字符的[迭代器](iterators.html),
> - 删除从start到end的所有字符, 返回一个[迭代器](iterators.html),指向被删除的最后一个字符的下一个位置
> - 删除从index索引开始的num个字符, 返回***this**.

    上面的程序里，用了第三条语法，就是删掉第一个字符。

    还有要注意的就是用vector申请数组时，如果要像这样初始化，那么一开始就不要申请固定的空间，不然会报错。

    分组这种方法，一开始还没看懂，一直纠结当长度为8或者4怎么办，看到后面发现，这种情况相当于直接跳过了。

    1,0000,5000，用上面的程序，这个数字输出是一亿五千，并没有错，我以为要读一亿零五千。
