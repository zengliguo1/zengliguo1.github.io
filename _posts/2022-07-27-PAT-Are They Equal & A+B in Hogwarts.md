---
title: PAT-Are They Equal & A+B in Hogwarts
date: 2022-07-27 23:11:00 +0800
categories: [算法刷题, PAT]
tags: [数学]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1060 [Are They Equal](https://pintia.cn/problem-sets/994805342720868352/problems/994805413520719872)

```cpp
//考虑前导0，整数部分、小数部分
#include<iostream>
#include<string>
using namespace std;
//将输入的数字转换成科学计数法的数字
void format(string &num, int N)
{
    int e = 0;//记录指数
    //首先判断先导0，并删除先导0
    while(num.size() > 0 && num[0] == '0')
    {
        num.erase(num.cbegin());//删掉第一位0
    }
    //删完先导0后，分为有小数点和没小数点的数
    if(num.find('.') == string::npos)//如果没有小数点
    {
        e = num.size();
    }
    else//有小数点，要么是123.456这样的数，要么是.123456
    {
        int pos = num.find('.');
        if(pos > 0)//如果小数点左边有数，就像第一种情况
        {
            e = pos;
            num.erase(num.cbegin() + pos);//删除小数点
        }
        else//如果小数点左边啥也没有
        {
            num.erase(num.cbegin());//删掉小数点
            //统计先导0，删除并记录更新e
            while(num.size() > 0 && num[0] == '0')
            {
                num.erase(num.cbegin());//删掉0
                e--;
            }
            //如果num给删空了，说明是.000000这样的数
            if(num.size() == 0) e = 0;
        }
    }
    //以上已经把num的有效位抽出来了，把先导0、小数点已经删除了
    //接下来就是取前N位，不够的补0
    if(num.size() > N)//说明当前有效位够了，取子串即可
    {
        num = num.substr(0, N);
    }
    else//需要补0
    {
        //需要补N - num.size()个0
        int add0 = N - num.size();
        for(int i = 0; i < add0; i++)
        {
            num.push_back('0');
        }
    }
    //更新为转换后的字符串
    num = "0." + num + "*10^" + to_string(e);
}

int main()
{
    //读入数据
    int N;
    string A, B;
    cin>>N>>A>>B;
    //转换成科学记数法
    format(A, N);
    format(B, N);
    if(A == B)
    {
        cout<<"YES "<<A;
    }
    else
    {
        cout<<"NO "<<A<<' '<<B;
    }
}
```

    这道题有一定的难度，需要考虑的情况比较多，整体思路就是将读入的数字转换成科学计数法的数字，再做比较，转换过程中直接对原字符串进行修改。

    开始有一半的错，是因为打印"NO"，我写成了"No"，改过来后，只有最后一个点没过，原因是补0的问题，在测**5 0 00.0**这个数据时，发现打印的是0.000，少打了2个0，这才锁定了问题的位置，然后发现，一开始我的补0是这么写的：

```cpp
        //需要补N - num.size()个0

        for(int i = 0; i  N - num.size(); i++)
        {
            num.push_back('0');
        }
```

    这就是问题所在，每次补0后，`num.size()`是会变大的，所以才会少0！这个问题其实以前也出过，没想到现在还会犯这种错误，真是不应该。所以以后要注意循环时的界。

## A1058 [A+B in Hogwarts](https://pintia.cn/problem-sets/994805342720868352/problems/994805416519647232)

```cpp
#include<iostream>
#include<cstdio>

using namespace std;

int main()
{
    long long ag,as,ak,bg,bs,bk;
    scanf("%lld.%lld.%lld %lld.%lld.%lld", &ag, &as, &ak,  &bg, &bs, &bk);
    long long sum = (ag + bg) * (29 * 17) + (as + bs) * 29 + (ak + bk);
    printf("%lld.%lld.%lld", sum / (29 * 17), sum % (29 * 17) / 29, sum % 29);
}
```

    这道题目其实挺简单的，但被我给弄复杂了。

    首先，用scanf读取会更方便；

    其次，要使用long long 不然会有测试点溢出。
