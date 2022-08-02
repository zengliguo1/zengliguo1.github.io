---
title: PAT-Recover the Smallest Number & Perfect Sequence & Insert or Merge & 二分查找函数upper_bound()
date: 2022-08-02 22:26:00 +0800
categories: [算法刷题, PAT]
tags: [贪心, 二分]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1038 [Recover the Smallest Number](https://pintia.cn/problem-sets/994805342720868352/problems/994805449625288704)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<algorithm>
using namespace std;

bool comp(string& s1, string& s2)
{
    /*
    int len = s1.size() > s2.size() ? s1.size() : s2.size();//取长度最长的来比较
    //短的string到头了就要从头接着比
    for (int j = 0; j < len; j++)
    {
        if (s1[j % s1.size()] == s2[j % s2.size()])//如果相同位相同
        {
            continue;
        }
        else//如果不同，小的在前面
        {
            return s1[j % s1.size()] < s2[j % s2.size()];
        }
    }
    //如果遍历到最后都相等，其实怎么排都无所谓了
    return true;*/
    return s1 + s2 < s2 + s1;//如果s1 + s2小，就把s1排前面
}

int main()
{
    int N;
    cin >> N;//数字片段个数
    vector<string> segs;
    for (int i = 0; i < N; i++)
    {
        string _num;
        cin >> _num;
        segs.emplace_back(_num);
    }
    //排序
    sort(segs.begin(), segs.end(), comp);
    //输出：记得去掉先导0
    bool zero;
    if (segs[0][0] == '0')zero = true;//说明有先导0
    else zero = false;
    for (int i = 0; i < segs.size(); i++)
    {
        if (zero)//如果当前还有先导0
        {
            for (int j = 0; j < segs[i].size(); j++)//遍历当前字符串的每个字符
            {
                if (segs[i][j] != '0')//如果已经不是0了，那就把剩下的输出
                {
                    zero = false;
                    cout << segs[i].substr(j);
                    break;
                }
            }
        }
        else//当前已经没有先导0
        {
            cout << segs[i];
        }
    }
    if (zero)//如果到最后了都是0
    {
        cout << 0;
    }
}
```

    这道题按我的程序做完后，有两个测试点（一共6分）过不去，其中一个是特判，如果号码全是0，那么要输出0，还有一处，其实是我comp函数写的不对（注释里的是我写的），我已经想到了两个长度不同的字符串顺序不同大小不同这一点，但我进行判断的方法很复杂，还不能保证一定对，因为我是按长串的长度来一一比对，短串到头了再重新比，如果都相同就return false（此时我觉得，return什么都行），但实际上不应该是这样，比如123和1231，按我的方法，这两个串就是相等了，谁放在前面都行，但1231123是小于1231231的，像我写的return false的话，123会放在1321的前面，这样答案就不对了，改成true可以通过这一个样例，但实际上还是不对的，如果样例多一点，肯定还是不行的。

    直接两串相加对比，既简单，又正确 。

## A1085 [Perfect Sequence](https://pintia.cn/problem-sets/994805342720868352/problems/994805381845336064)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
typedef  long long LL;
int main()
{
    LL N, p;
    cin >> N >> p;//序列中元素个数N，参数p
    vector<LL> sequence;
    //读取每个数
    for(int i = 0; i < N; i++)
    {
        LL _num;
        cin >> _num;
        sequence.emplace_back(_num);
    }
    sort(sequence.begin(), sequence.end());
    LL goal; 
    int ans = 0;
    //从最小的开始遍历，二分查找右侧第一个大于m*p并返回它的位置
    for (int i = 0; i < N; i++)
    {
        goal = sequence[i] * p;
        //pos为第一个大于m*p的位置，pos-i处的位置正好是等于序列长
        auto pos = upper_bound(sequence.begin(), sequence.end(), goal);
        int len = pos - (sequence.begin() + i);
        if (len > ans)
        {
            ans = len;
        }
    }
    cout << ans;
}
```

    这道题目其实从某种意义上来讲是暴力算法，只是用二分把复杂度从$O(n^2)$降到了$O(nlog(n))$，其中`upper_bound()`函数就是二分查找，非常方便，记录下

## upper_bound() & lower_bound()

* upper_bound(I first, S last, const T& value)

> 指向首个*大于* `value` 的元素的迭代器，或若找不到这种元素则为 `last` 。

* lower_bound( ForwardIt first, ForwardIt last, const T& value );

> 指向首个*不小于* `value` 的元素的迭代器，或若找不到这种元素则为 `last` 。

## A1089 [Insert or Merge](https://pintia.cn/problem-sets/994805342720868352/problems/994805377432928256)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;


bool isSame(vector<int>& tmp, vector<int>& partial)
{
    for (int i = 0; i < tmp.size(); i++)
    {
        if (tmp[i] != partial[i]) return false;
    }
    return true;
}
bool insertSort(vector<int>& origin, vector<int>& partial)
{
    bool flag = false;//标记是否和部分排序的数组一样了
    for (int i = 1; i < origin.size(); i++)//进行size()-1次排序
    {
        //先判断是否和部分排序的数组一样了
        if (i != 1 && isSame(origin, partial))//初始数组不进行判断
        {
            flag = true;
        }
        //插入排序
        int tmp = origin[i];//储存当前要插入的元素
        int j = i;
        while (j > 0 && origin[j - 1] > tmp)//如果前面一个元素大于要插入的元素
        {
            origin[j] = origin[j - 1];
            j--;
        }
        origin[j] = tmp;
        //判断是否可以退出了
        if (flag)//此时说明是插入排序，而且下一步也做好了
        {
            return true;
        }
    }
    return false;//排完了还不相等
}

void mergeSort(vector<int>& origin, vector<int>& partial)
{
    bool flag = false;
    //以2为步长，每次翻倍
    for (int step = 2; step / 2 < origin.size(); step *= 2)
    {
        //先判断是否和部分排序数组一致
        if (isSame(origin, partial))
        {
            flag = true;
        }
        //归并排序
        for (int i = 0; i < origin.size(); i += step)
        {
            sort(origin.begin() + i, min(origin.begin() + i + step, origin.end()));
        }
        //如果一致了
        if (flag)
        {
            return;
        }
    }
}

void showArray(vector<int> &array)
{
    for (int i = 0; i < array.size(); i++)
    {
        if (i != 0)
        {
            cout << ' ' << array[i];
        }
        else
        {
            cout << array[i];
        }
    }
}
int main()
{
    int N;//序列数字个数
    cin >> N;
    vector<int> origin(N);
    vector<int> partial(N);
    for (int i = 0; i < N; i++)
    {
        cin >> origin[i];
    }
    for (int i = 0; i < N; i++)
    {
        cin >> partial[i];
    }
    vector<int> tmp(origin);
    if (insertSort(tmp, partial))
    {
        cout << "Insertion Sort" << endl;
        showArray(tmp);
    }
    else
    {
        mergeSort(origin, partial);
        cout << "Merge Sort" << endl;
        showArray(origin);
    }
}
```

    这道题对我来说稍有难度，涉及到插入排序和归并排序的具体书写，其中归并排序的迭代写法通过每次迭代step的方式来循环。插入排序中，插入时，从数组排好序的尾部开始比对，直到比对到0或者小于要插入元素时才停止。
