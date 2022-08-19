---
title: PAT-Rational Arithmetic & Prime Factors
date: 2022-08-19 22:05:00 +0800
categories: [算法刷题, PAT]
tags: [数学, 最大公因数， 最小公倍数]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1088 [Rational Arithmetic](https://pintia.cn/problem-sets/994805342720868352/problems/994805378443755520)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<string>
using namespace std;
typedef long long LL;
vector<LL> numerator(2);
vector<LL> denominator(2);
LL gcd(LL a, LL b)
{
    if (a < 0) a *= -1;
    if (b < 0) b *= -1;
    if (b == 0) return a;
    else return gcd(b, a % b);
}
LL lcm(LL a, LL b)
{
    if (a < 0) a *= -1;
    if (b < 0) b *= -1;
    return a * (b / gcd(a, b));
}

LL ansNu, ansDe, a;
string sAns;
void Plus()
{
    a = 0;
    ansDe = lcm(denominator[0], denominator[1]);
    ansNu = numerator[0] * (ansDe / denominator[0]) + numerator[1] * (ansDe / denominator[1]);
    LL gc = gcd(ansNu, ansDe);
    ansNu /= gc;
    ansDe /= gc;
    a = ansNu / ansDe;
}
void Minus()
{
    ansDe = lcm(denominator[0], denominator[1]);
    ansNu = numerator[0] * (ansDe / denominator[0]) - numerator[1] * (ansDe / denominator[1]);
    LL gc = gcd(ansNu, ansDe);
    ansNu /= gc;
    ansDe /= gc;
    a = ansNu / ansDe;
}
void Multi()
{
    ansDe = denominator[0] * denominator[1];
    ansNu = numerator[0] * numerator[1];
    LL gc = gcd(ansNu, ansDe);
    ansNu /= gc;
    ansDe /= gc;
    a = ansNu / ansDe;
}
void Divide()
{
    if (numerator[1] == 0)
    {
        sAns = "Inf";
        return;
    }
    ansDe = denominator[0] * numerator[1];
    ansNu = numerator[0] * denominator[1];
    if (ansDe < 0)
    {
        ansDe *= -1;
        ansNu *= -1;
    }
    LL gc = gcd(ansNu, ansDe);
    ansNu /= gc;
    ansDe /= gc;
    a = ansNu / ansDe;
}
void Format()
{
    if (sAns == "Inf") return;
    if (ansNu % ansDe == 0) sAns = to_string(a);//整数
    else if (abs(ansNu) < ansDe)//真分数
    {
        sAns = to_string(ansNu) + "/" + to_string(ansDe);
    }
    else//假分数
    {
        sAns = to_string(a) + " " + to_string(abs(ansNu) % ansDe) + "/" + to_string(ansDe);
    }

    if (a < 0 || ansNu < 0)
    {
        sAns = "(" + sAns + ")";
    }
}
int main()
{
    scanf("%lld/%lld %lld/%lld", &numerator[0], &denominator[0], &numerator[1], &denominator[1]);
    //格式化左式
    string s[2];
    for (int i = 0; i < 2; i++)
    {
        if (numerator[i] % denominator[i] == 0)//整数
        {
            s[i] = to_string(numerator[i] / denominator[i]);
        }
        else if(abs(numerator[i]) < denominator[i])//真分数
        {
            LL gc = gcd(numerator[i], denominator[i]);
            numerator[i] /= gc;
            denominator[i] /= gc;
            s[i] = to_string(numerator[i]) + "/" + to_string(denominator[i]);
        }
        else //假分数
        {
            LL gc = gcd(numerator[i], denominator[i]);
            numerator[i] /= gc;
            denominator[i] /= gc;
            s[i] = to_string(numerator[i] / denominator[i]) + " " + to_string(abs(numerator[i]) % denominator[i]) + "/" + to_string(denominator[i]);
        }
        if (numerator[i] < 0)//负数
        {
            s[i] = "(" + s[i] + ")";
        }
    }

    Plus();
    Format();
    cout << s[0] << " + " << s[1] << " = " << sAns << endl;

    Minus();
    Format();
    cout << s[0] << " - " << s[1] << " = " << sAns << endl;

    Multi();
    Format();
    cout << s[0] << " * " << s[1] << " = " << sAns << endl;

    Divide();
    Format();
    cout << s[0] << " / " << s[1] << " = " << sAns << endl;

}
```

    这道题写的蛮久的，开始只拿了17min，去查了测试点2才重新拿到满分，注意的点：

* 一开始有个浮点错误，那是因为忘了特判除数为0输出Inf了

* 起初调试时，发现除法不对劲，分母出现了负数，加个判断即可

* 最后是测试点2答案错误，去牛客测试了下（会给错误样例是什么），发现是在等式左侧的不仅整数要化简，能约分的也要约分

* 算最大公约数和最小公倍数时，要注意参数都应该是正的

## A1059 [Prime Factors](https://pintia.cn/problem-sets/994805342720868352/problems/994805415005503488)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
#include<unordered_map>
using namespace std;
typedef long long LL;
const LL maxn = 10010;
vector<LL> primeFactors;
unordered_map<LL, int> map;
int N;
bool isPrime(LL n)
{
	//LL sqr = sqrt(1.0 * n);
	for (LL i = 2; i * i<= n; i++)
	{
		if (n % i == 0) return false;
	}
	return true;
}
void Find()
{
	//LL sqr = sqrt(1.0 * N);
	for (LL i = 2; i * i < N; i++)
	{
		if (N % i == 0 && isPrime(i))
		{
			primeFactors.emplace_back(i);
			map[i]++;
		}
	}
}

int main()
{
	cin >> N;
	LL ansN = N;
	Find();
	if (primeFactors.size() == 0)
	{
		cout << ansN << '=' << ansN;
		return 0;
	}
	for (int i = 0; i < primeFactors.size(); i++)
	{
		N /= primeFactors[i];
	}
	int j = 0;
	while (N != 1)
	{
		while (N % primeFactors[j] == 0)
		{
			map[primeFactors[j]]++;
			N /= primeFactors[j];
		}
		j++;
	}
	cout << ansN << '=';
	for (int i = 0; i < primeFactors.size(); i++)
	{
		if (i == 0)
		{
			cout << primeFactors[i];
		}
		else
		{
			cout << '*' << primeFactors[i];
		}
		if (map[primeFactors[i]] > 1)
		{
			cout << '^' << map[primeFactors[i]];
		}
	}
	return 0;
}
```

    32分钟拿到21分（满分25），一个答案错误，一个段错误，搜了下，其实就一个问题，如果输入的数本身就没有质因数，那么就输出等于自己就行了，在pat这里可以拿满分，但在牛客那里还有几个浮点错误，其实就是我设置的find，是小于`sqrt（N）`了，如果小于N的话能找到，但会超时，也就是说我这个方法并不好。

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
#include<unordered_map>
using namespace std;
typedef long long LL;
const LL maxn = 12000000;
vector<LL> prime;
vector<LL> factors;
bool p[maxn] = { false };
LL n;
void Era()//使用欧拉筛法算出素数表
{
	for (LL i = 2; i < maxn; i++)
	{
		if (p[i] == false)
		{
			prime.emplace_back(i);
			for (int j = i * 2; j < maxn; j += i)
			{
				p[j] = true;
			}
		}
	}
}

int main()
{
	Era();
	cin >> n;
	unordered_map<LL, int> map;
	for (LL p : prime)
	{
		if (n % p == 0)
		{
			map[p]++;
			factors.emplace_back(p);
		}
	}
	if (map.empty())
	{
		cout << n << '=' << n;
		return 0;
	}
	LL ansN = n;
	for (LL f : factors)
	{
		n /= f;
	}
	int j = 0;
	while (n != 1)
	{
		while (n % factors[j] == 0)
		{
			n /= factors[j];
			map[factors[j]]++;
		}
		j++;
	}
	cout << ansN << '=';
	for (int i = 0; i < factors.size(); i++)
	{
		if (i == 0)
		{
			cout << factors[i];
		}
		else
		{
			cout << '*' << factors[i];
		}
		if (map[factors[i]] > 1)
		{
			cout << '^' << map[factors[i]];
		}
	}
	return 0;
}
```

    使用欧拉筛法，先算出素数表，然后再算，就可以用过牛客的所有样例了，牛客是真狠，最大到一千多万了，得设置`maxn`为很大很大，不用欧拉筛法估计还得超时，pat数据没那么过分。

* 注意，如果要使用`vector`的`emplace_back`函数，那就不要初始化空间，不然无效（但不报错，也不中断，怪怪的）
