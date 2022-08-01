---
title: PAT-Be Unique & String Subtraction & Find Coins & Mooncake & Magic Coupon & Sort with Swap(0, i)
date: 2022-08-01 21:56:00 +0800
categories: [算法刷题, PAT]
tags: [哈希表, 贪心]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1041 [Be Unique](https://pintia.cn/problem-sets/994805342720868352/problems/994805444361437184)

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
using namespace std;

int main()
{
    unordered_map<int, int> map;
    int N;//数字个数
    cin >> N;
    vector<int> nums;//按顺序储存所有的数字
    for (int i = 0; i < N; i++)
    {
        int num;
        cin >> num;
        if (!map.count(num))//如果之前没遇到过num
        {
            nums.emplace_back(num);
        }
        map[num]++;
    }
    for (int n : nums)
    {
        if (map[n] == 1)
        {
            cout << n;
            return 0;
        }
    }
    cout << "None";
    return 0;
}
```

    简单题，不多解释。

## A1050 [String Subtraction](https://pintia.cn/problem-sets/994805342720868352/problems/994805429018673152)

```cpp
#include<iostream>
#include<string>
#include<unordered_map>
using namespace std;

int main()
{
    unordered_map<char, int> map;
    string s1, s2;
    getline(cin, s1);
    getline(cin, s2);
    for (char c : s2)
    {
        map[c]++;
    }
    for (char c : s1)
    {
        if (!map.count(c))
        {
            cout << c;
        }
    }
}
```

    简单的哈希表

## A1048 [Find Coins](https://pintia.cn/problem-sets/994805342720868352/problems/994805432256675840)

```cpp
#include<iostream>
#include<string>
#include<unordered_map>
#include<set>
using namespace std;

int main()
{
    int N, M;//硬币数、金额
    cin >> N >> M;
    unordered_map<int, int> coinMap;
    set<int> coinKind;
    for (int i = 0; i < N; i++)
    {
        int _coin;
        cin >> _coin;
        coinKind.insert(_coin);
        coinMap[_coin]++;
    }
    for (auto c : coinKind)
    {
        if (2 * c > M) break;//如果c大于M的一半，直接退出循环
        else if (2 * c == M)//如果c是M的一半
        {
            if (coinMap[c] >= 2)
            {
                cout << c << ' ' << M - c;
                return 0;
            }
        }
        else if (coinMap.count(M - c))
        {
            cout << c << ' ' << M - c;
            return 0;
        }
    }
    cout << "No Solution";
    return 0;
}
```

    这道题有个小陷阱，就是使用的硬币V1得<=V2，其中如果等于的话，得看map里有没有两个等面值的硬币。

## A1070 [Mooncake](https://pintia.cn/problem-sets/994805342720868352/problems/994805399578853376)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
struct mooncake
{
    double inventory;//库存
    double totalPrice;//总价
    double price;//单价
    mooncake(double _inventory) :inventory(_inventory) {}
};
bool comp(mooncake& c1, mooncake& c2)
{
    return c1.price > c2.price;
}
int main()
{
    int N, D;//月饼种类、超商需求
    cin >> N >> D;
    double demand = (double)D;
    vector<mooncake> cakes;
    //读取每种月饼的库存
    for (int i = 0; i < N; i++)
    {
        double _inventory;
        cin >> _inventory;
        cakes.emplace_back(mooncake(_inventory));
    }
    for (int i = 0; i < N; i++)
    {
        double _price;
        cin >> _price;
        cakes[i].totalPrice = _price;
        cakes[i].price = _price / cakes[i].inventory;
    }
    //按价位排序
    sort(cakes.begin(), cakes.end(), comp);
    double profit = 0.0;
    for (auto c : cakes)
    {
        if (c.inventory < demand)//如果当前种类月饼的库存小于需求，那么全买
        {
            profit += c.totalPrice;
            demand -= c.inventory;
        }
        else if (c.inventory == demand)//如果当前种类月饼的库存等于需求，也全买并且退出
        {
            profit += c.totalPrice;
            break;
        }
        else//如果当前种类月饼的库存大于需求，只买一部分并且退出
        {
            profit += ((double)demand / (double)c.inventory * c.totalPrice);
            break;
        }
    }
    printf("%.2lf", profit);
}
```

    这道题，外表看似简单，其实隐藏了一个惊天大陷阱。注意input Specification中是这么讲的：the first line contains 2 **positive integers**；Then the second line gives the **positive inventory amounts**，这后面的库存其实只是正数，没说不能是小数，所以如果都按整型来算，是会有一个测试点过不去的，岂可休！

## A1037 [Magic Coupon](https://pintia.cn/problem-sets/994805342720868352/problems/994805451374313472)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

bool comp1(int a, int b)//从大到小排序
{
    return a > b;
}
int main()
{
    int nC, nP;//优惠券数量，产品数量
    vector<int> couponPostive, couponNegtive;//分别储存正、负优惠券
    vector<int> productPostive, productNegtive;//分别储存正、负产品
    cin >> nC;
    while (nC--)
    {
        int c;
        cin >> c;
        //只考虑正负，不考虑0
        if (c > 0) couponPostive.emplace_back(c);
        else if (c < 0) couponNegtive.emplace_back(c);
    }
    cin >> nP;
    while (nP--)
    {
        int p;
        cin >> p;
        if (p > 0) productPostive.emplace_back(p);
        else if (p < 0) productNegtive.emplace_back(p);
    }
    //给正数数组按从大到小排序
    sort(couponPostive.begin(), couponPostive.end(), comp1);
    sort(productPostive.begin(), productPostive.end(), comp1);
    //给负数数组按从小到大排序
    sort(couponNegtive.begin(), couponNegtive.end());
    sort(productNegtive.begin(), productNegtive.end());
    int maximum = 0;
    //计算正数中的最大收益
    //取二者小的
    int len = couponPostive.size() < productPostive.size() ? couponPostive.size() : productPostive.size();
    for (int i = 0; i < len; i++)
    {
        maximum += (couponPostive[i] * productPostive[i]);
    }
    //同样取二者小的
    len = couponNegtive.size() < productNegtive.size() ? couponNegtive.size() : productNegtive.size();
    for (int i = 0; i < len; i++)
    {
        maximum += (couponNegtive[i] * productNegtive[i]);
    }
    cout << maximum;
}
```

    简单的贪心算法。

## A1067 [Sort with Swap(0, i)](https://pintia.cn/problem-sets/994805342720868352/problems/994805403651522560)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

int main()
{
    int N;//序列长度
    cin >> N;
    vector<int> pos(N);
    //记录每个数字的位置
    int mismatch = 0;//不在本位个数（除了0）
    for (int i = 0; i < N; i++)
    {
        int _num;
        cin >> _num;
        pos[_num] = i;
        if (pos[_num] != _num && _num != 0)//统计有多少个数（除了0）不在本位
        {
            mismatch++;
        }
    }
    //贪心算法，硬贪，先判断0在不在0位上，不在的话就直接交换，
    //在的话就跟第一个不在本位的数字交换后再换
    int ans = 0;
    int k = 1;//记录不在本位数字的最小下标/位置
    while (mismatch > 0)//还有数字不在本位
    {
        //如果0在位置0
        if (pos[0] == 0)
        {
            //那么去找一个不在本位的数字，和它交换位置
            while (k < N && pos[k] == k)
            {
                k++;
            }
            swap(pos[0], pos[k]);//0和k的位置互换
            ans++;
        }
        while (pos[0] != 0)//如果0不在位置0
        {
            //正常交换
            swap(pos[0], pos[pos[0]]);
            ans++;
            mismatch--;
        }
    }
    cout << ans;
}
```

    这道题，说实话，有点小难，虽然我想到了这个策略，但我觉得并不一定真的是最优的策略，就止步不前了，没想到还就是这么个策略，看来贪心算法真是要有胆量去做。

    直接默认如果0到了0位却还没有都归位，那就让0和最邻近的一个没归位的数交换位置。
