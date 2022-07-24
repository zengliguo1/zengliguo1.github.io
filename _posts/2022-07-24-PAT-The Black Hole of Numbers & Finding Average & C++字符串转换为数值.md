---
title: PAT-The Black Hole of Numbers & Finding Average & C++字符串转换为数值
date: 2022-07-24 14:52:00 +0800
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

## A1069 [The Black Hole of Numbers](https://pintia.cn/problem-sets/994805342720868352/problems/994805400954585088)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<cstdio>
#include<algorithm>
using namespace std;
bool cmp(char &a, char &b)
{
    return a > b;
}
int main()
{
    string curNum;//记录当前数字
    cin>>curNum;

    while(1)
    {
        while(curNum.size() < 4)
        {
            curNum = "0" + curNum;
        }
        //开始做减法
        sort(curNum.begin(), curNum.end(), cmp);
        int a = stoi(curNum);
        sort(curNum.begin(), curNum.end());//升序
        int b = stoi(curNum);
        int c = a - b;//算差
        if(c == 0)
        {
            printf("%04d - %04d = 0000\n", stoi(curNum), stoi(curNum));
            break;
        }
        else
        {
            printf("%04d - %04d = %04d\n", a, b, c);
        }
        //判断是不是=6174
        if(c == 6174) break;
        curNum = to_string(c);//更新当前数字
    }
}
```

    这道题虽然是个数学，但也是个模拟，也就是说，题目说是四位数，那么在运算，也就是排序的时候也要按四位数来，不满四位就补0，不然排序后的数是不对的。

## [C++字符串转换为数值](https://blog.csdn.net/sinat_40872274/article/details/81367815)

## A1108 [Finding Average](https://pintia.cn/problem-sets/994805342720868352/problems/994805360777347072)

```cpp
#include<iostream>
#include<sstream>

using namespace std;
bool isValid(string &str)
{
    //如果输入的字符串中，包含英文、包含多个小数点、精度大于2，都是非法的
    int i = 0;
    //如果第一位是-号
    if(str[0] == '-') i++;
    //如果除去-号，第一位还是符号而且不是.，那么返回false
    if(!isdigit(str[i]) && str[i] != '.')return false;
    //先遍历整数部分
    for(; i < str.size(); i++)
    {
        if(str[i] == '.')
        {
            i++;
            break;//遇到第一个点就中止
        }
        if(!isdigit(str[i])) return false;//如果遇到的不是点而是其他非数字
    }
    //遍历小数部分
    int remainder = str.size() - i;//小数点后面的位数
    if(remainder > 2) return false;//如果小数点后面超过两位
    for(; i < str.size(); i++)
    {
        if(!isdigit(str[i])) return false;
    }
    return true;
}

int main()
{
    int N;
    cin>>N;
    int cnt = 0;
    double sum = 0;
    while(N--)
    {
        string str;
        cin>>str;
        if(isValid(str) && stod(str) >= -1000 && stod(str) <= 1000)
        {
            sum += stod(str);
            cnt++;
        }
        else
        {
            cout<<"ERROR: "<<str<<" is not a legal number"<<endl;
        }
    }
    if(cnt == 0)
    {
        cout<<"The average of 0 numbers is Undefined";
    }
    else if(cnt == 1)
    {
        printf("The average of 1 number is %.2lf", sum);
    }
    else
    {
        printf("The average of %d numbers is %.2lf", cnt, sum/cnt);
    }
}
```

    这道题太搞人了哈哈哈哈，刚开始有半测试点过不去，是因为考虑的情况太少，只考虑了的字符串中，包含英文、包含多个小数点的情况，其他情况没考虑到，后面又改了改isValid的函数，但始终有一个点过不去，我去查了查，才知道，当只有一个number的时候，不能打印1 numbers，不然会报错，好家伙，这真的只是考验代码水平的吗233333。

    还有一个要注意的是，在算精度超过2位的情况时，5给我判断成超过两位了，debug了之后发现`if(str.size() - i > 2) return false;`这么写竟然会返回false，明明`str.size() - i = -1`，后来单独先赋给一个变量，再用变量去做比较，就比较对了，很怪。

    我又去debug了一下，发现了，因为`str.size()`的数据类型是`unsigned long long`，`str.size() - i`默认的数据类型是无符号longlong型，所以-1其实是18446744073709551615，怪不得会误判。
