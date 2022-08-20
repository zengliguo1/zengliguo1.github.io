---
title: PAT-Hashing & Consecutive Factors & Come on! Let's C & A Delayed Palindrome
date: 2022-08-20 17:09:00 +0800
categories: [算法刷题, PAT]
tags: [数学, 素数, 因式分解, 大整数加法]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1078 [Hashing](https://pintia.cn/problem-sets/994805342720868352/problems/994805389634158592)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
#include<unordered_map>
using namespace std;
const int maxn = 100000;
bool primeTable[maxn] = {false};
vector<int> prime;
int m, n;
void Find()
{
    for (int i = 2; i < maxn; i++)
    {
        if (primeTable[i] == false)
        {
            prime.emplace_back(i);
            for (int j = i * 2; j < maxn; j += i)
            {
                primeTable[j] = true;
            }
        }
    }
}
bool isPrime(int n)
{
    if (n <= 1) return false;
    int sqr = (int)sqrt(1.0 * n);
    for (int i = 2; i <= sqr; i++)
    {
        if (n % i == 0) return false;
    }
    return true;
}
int main()
{
    Find();
    cin >> m >> n;
    int t;
    if (isPrime(m)) t = m;
    else
    {
        for (int i = 0; i < prime.size(); i++)
        {
            if (prime[i] >= m)
            {
                t = prime[i];
                break;
            }
        }
    }

    unordered_map<int, int> map;
    for (int i = 0; i < n; i++)
    {
        int num;
        cin >> num;
        if (i != 0)cout << ' ';
        if (map.count(num % t))
        {
            //二次方探查法
            bool flag = false;
            for (int j = 1; j < t; j++)
            {
                int pos = (num + j * j) % t;
                if (!map.count(pos))
                {
                    map[pos]++;
                    cout << pos;
                    flag = true;
                    break;
                }
            }
            if (flag == false) cout << '-';
        }
        else
        {
            cout << num % t;
            map[num % t]++;
        }
    }
}
```

    题目中**Quadratic probing (with positive increments only) is used to solve the collisions.** 这句话没看懂，是二次方探查法（只考虑正向探查）。12分钟20分（满分25），也还凑合，加上二次方探查法就满分了，pat真的带善人。注意：

* isPrime函数中一定要记得特判，如果n小于等于1，那就返回false，不然1会判断为合数

## A1096 [Consecutive Factors](https://pintia.cn/problem-sets/994805342720868352/problems/994805370650738688)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cmath>
#include<unordered_map>
using namespace std;
typedef long long LL;
LL n;
bool isPrime(LL num)
{
    if (num <= 1) return false;
    LL sqr = (LL)sqrt(1.0 * num);
    for (LL i = 2; i <= sqr; i++)
    {
        if (num % i == 0) return false;
    }
    return true;
}

int main()
{
    cin >> n;
    if (isPrime(n))
    {
        cout << 1 << endl << n;
        return 0;
    }
    LL sqr = (LL)sqrt(1.0 * n);
    LL ansIndex, ansLen = 0;
    for (LL i = 2; i <= sqr; i++)
    {
        LL j = i, tmp = i;
        while (n % tmp == 0)
        {
            j++;
            tmp *= j;
        }
        if (j - i > ansLen)
        {
            ansLen = j - i;
            ansIndex = i;
        }
    }
    cout << ansLen << endl;
    for (int i = 0; i < ansLen; i++)
    {
        if (i != 0) cout << '*';
        cout << ansIndex;
        ansIndex++;
    }
}
```

    这道题我陷入了一个误区，由于之前做过一道分解质因数，因为给的数肯定时会被分解完的，所以就可以记录每一个质因数，但这道题并不是分解质因数，而是连续的因数，我此时就在想，这样的话分解的情况有很多，并不能把数分解完了再去序列里去查，不是质因数，这怎么分解，十分复杂。这就是我陷入的误区。实际上并不需要把一个数的所有因子分解出来，只需要查哪一段连续的序列乘积正好可以被原数整除即可，这样就方便多了。当然需要特判那些质数。

## A1116 [Come on! Let's C](https://pintia.cn/problem-sets/994805342720868352/problems/994805355358306304)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<string>
#include<unordered_map>
using namespace std;

const int maxn = 20000;
vector<bool> p(maxn, false);
unordered_map<string, string> map;
unordered_map<string, int> check;
unordered_map<int, int> prime;
int n, q;
void Era()
{
	for (int i = 2; i < maxn; i++)
	{
		if (p[i] == false)
		{
			prime[i]++;
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
	for (int i = 1; i <= n; i++)
	{
		string id;
		cin >> id;
		if (i == 1)
		{
			map[id] = "Mystery Award";
		}
		else if (prime.count(i))
		{
			map[id] = "Minion";
		}
		else
		{
			map[id] = "Chocolate";
		}
	}
	cin >> q;
	for (int i = 0; i < q; i++)
	{
		string id;
		cin >> id;
		if (!map.count(id))
		{
			cout << id << ": Are you kidding?" << endl;
			continue;
		}
		if (check.count(id))
		{
			cout << id << ": Checked" << endl;
		}
		else
		{
			check[id]++;
			cout << id << ": " << map[id] << endl;
		}
	}
}

```

    素数判定+哈希表。注意如果询问的id不存在的情况。16分钟拿下。

## A1136 [A Delayed Palindrome](https://pintia.cn/problem-sets/994805342720868352/problems/994805345732378624)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<cstring>
using namespace std;

const int maxn = 1010;
struct bign
{
	int d[maxn];
	int len;
	bign()
	{
		memset(d, 0, sizeof(d));
		len = 0;
	}
};
void exchange(bign& a, string& b)
{
	a.len = b.size();
	for (int i = 0; i < a.len; i++)
	{
		a.d[i] = b[b.size() - 1 - i] - '0';
	}
}
bign ReverseAdd(bign& bn)
{
	int len = bn.len;
	bign bn2;
	bn2.len = len;
	for (int i = 0; i < len; i++)
	{
		bn2.d[i] = bn.d[len - 1 - i];
	}
	bign bn3;
	bn3.len = len;
	int carray = 0;
	for (int i = 0; i < len; i++)
	{
		int cur = bn.d[i] + bn2.d[i] + carray;
		carray = cur / 10;
		cur %= 10;
		bn3.d[i] = cur;
	}
	if (carray > 0)
	{
		bn3.d[len] = carray;
		bn3.len++;
	}
	for (int i = len - 1; i >= 0; i--)
	{
		cout << bn.d[i];
	}
	cout << " + ";
	for (int i = len - 1; i >= 0; i--)
	{
		cout << bn2.d[i];
	}
	cout << " = ";
	for (int i = bn3.len - 1; i >= 0; i--)
	{
		cout << bn3.d[i];
	}
	cout << endl;
	return bn3;
}
bool isPalin(bign& bn)
{
	for (int i = 0; i < bn.len / 2; i++)
	{
		if (bn.d[i] != bn.d[bn.len - 1 - i]) return false;
	}
	return true;
}
int main()
{
	string num;
	cin >> num;
	bign bn;
	exchange(bn, num);
	if (isPalin(bn))
	{
		for (int j = bn.len - 1; j >= 0; j--)
		{
			cout << bn.d[j];
		}
		cout << " is a palindromic number.";
		return 0;
	}
	for (int i = 0; i < 10; i++)
	{
		bn = ReverseAdd(bn);
		if (isPalin(bn))
		{
			for (int j = bn.len - 1; j >= 0; j--)
			{
				cout << bn.d[j];
			}
			cout << " is a palindromic number.";
			return 0;
		}
	}
	cout << "Not found in 10 iterations.";
}
```

    31分50秒拿下，说实话有点慢了，需要注意的点：

* 要先判断给的数是不是回文数，是的话就不用反转相加了，不然就只能拿14分

* 注意开辟数组的大小，不能是1000，否则最后一个测试点过不去，我估计是最高位进1了，到了1001位。
