---
title: PAT-Sum of Number Segments & Counting Ones & Rational Sum & Scanf()输入格式表
date: 2022-08-18 21:47:00 +0800
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

## A1104 [Sum of Number Segments](https://pintia.cn/problem-sets/994805342720868352/problems/994805363914686464)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

const int maxn = 100010;
vector<long double> num(maxn);
vector<long double> dfsSum(maxn);
int n;
long double sum = 0.0;

long double dfs(int i)
{
    if (i >= n) return 0;
    dfsSum[i] = (n - i) * num[i] + dfs(i + 1);
    return dfsSum[i];
}
int main()
{

    scanf("%d", &n);
    for (int i = 0; i < n; i++)
    {
        //cin >> num[i];
        scanf("%Lf", &num[i]);
    }
    dfs(0);
    for (int i = 0; i < n; i++)
    {
        sum += dfsSum[i];
    }
    printf("%.2Lf", sum);
}
```

    这道题有点小坑，我是当dfs做了，但网上很多都是按找规律做的（其实一开始我也想找规律，但刚列了两组数据就懒得列了，就去写dfs了），一开始就纯暴力，后面两个测试点超时，后来我发现又很多重复计算，于是就打表，虽然没有超时了，但测试点2过不去，于是开始查，发现是double的精度问题，改成long double就行了。

* scanf和printf读取打印long double类型，要写成`%Lf`，不然读不了也打不出，或者干脆用cin、cout（一开始超时，我就换了，但问题不在这里）

## A1049 [Counting Ones](https://pintia.cn/problem-sets/994805342720868352/problems/994805430595731456)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

int numOne(int num)
{
    int cnt = 0;
    while (num != 0)
    {
        if (num % 10 == 1)
        {
            cnt++;
        }
        num /= 10;
    }
    return cnt;
}
int n;
int main()
{
    int sum = 0;
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        sum += numOne(i);
    }
    cout << sum;
}
```

    直接暴力从1开始判断，这样能拿22分（满分30），其实也可以了，一共写了12分钟（其实有点长了，因为一开始还是花了点时间思考怎么才能不超时）

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

int main()
{
    int now, left, right, sum = 0;
    int a = 1;
    int n;
    cin >> n;
    while (n / a != 0)
    {
        left = n / (a * 10);
        right = n % a;
        now = (n / a) % 10;
        if (now == 0) sum += left * a;
        else if (now == 1)sum += left * a + right + 1;
        else sum += (left + 1) * a;
        a *= 10;
    }
    cout << sum;
}
```

    通过计算每一位是1的情况个数，可以通过全部样例，思路来自于算法笔记，真考到这种题目，就是可能得思考一会。

## A1081 [Rational Sum](https://pintia.cn/problem-sets/994805342720868352/problems/994805386161274880)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;
const int maxn = 110;
vector<long long> numerator(maxn);
vector<long long> denominator(maxn);
long long gcd(long long a, long long b)
{
	if (b == 0)return a;
	else return gcd(b, a % b);
}
long long lcm(long long a, long long b)
{
	return a * b / gcd(a, b);
}
void Plus(long long a, long long b)
{
	long long de = lcm(abs(denominator[a]), abs(denominator[b]));
	long long nu = numerator[a] * (de / denominator[a]) + numerator[b] * (de / denominator[b]);
	long long g = gcd(de, abs(nu));
	numerator[b] = nu / g;
	denominator[b] = de / g;
}

int n;
int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		scanf("%lld/%lld", &numerator[i], &denominator[i]);
	}
	for (int i = 0; i < n - 1; i++)
	{
		Plus(i, i + 1);
	}
	if (numerator[n - 1] % denominator[n - 1] == 0)
	{
		cout << numerator[n - 1] / denominator[n - 1];
	}
	else if (numerator[n - 1] >= denominator[n - 1])
	{
		cout << numerator[n - 1] / denominator[n - 1] << ' ' << numerator[n - 1] - (denominator[n - 1] * (numerator[n - 1] / denominator[n - 1])) << '/' << denominator[n - 1];
	}
	else
	{
		cout << numerator[n - 1] << '/' << denominator[n - 1];
	}
}
```

    写了最大公约数和最小公倍数的函数，这个记住比较好，开始有测试点过不去，把所有变量类型改为`long long`即可。

## Scanf()输入格式表

| 格式控制符           | 说明                                                            |
| --------------- | ------------------------------------------------------------- |
| %c              | 读取一个单一的字符                                                     |
| %hd、%d、%ld、%lld | 读取一个十进制整数，并分别赋值给 short、int、long、long long 类型                  |
| %ho、%o、%lo      | 读取一个八进制整数（可带前缀也可不带），并分别赋值给 short、int、long 类型                  |
| %hx、%x、%lx      | 读取一个十六进制整数（可带前缀也可不带），并分别赋值给 short、int、long 类型                 |
| %hu、%u、%lu      | 读取一个无符号整数，并分别赋值给 unsigned short、unsigned int、unsigned long 类型 |
| %f、%lf、%Lf      | 读取一个十进制形式的小数，并分别赋值给 float、double、long double 类型               |
| %e、%le          | 读取一个指数形式的小数，并分别赋值给 float、double 类型                            |
| %g、%lg          | 既可以读取一个十进制形式的小数，也可以读取一个指数形式的小数，并分别赋值给 float、double 类型         |
| %s              | 读取一个字符串（以空白符为结束）                                              |






