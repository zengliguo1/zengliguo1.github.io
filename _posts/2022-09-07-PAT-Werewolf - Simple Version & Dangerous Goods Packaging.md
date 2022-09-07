---
title: PAT-Werewolf - Simple Version & Dangerous Goods Packaging
date: 2022-09-07 12:13:00 +0800
categories: [算法刷题, PAT]
tags: [模拟, 哈希表]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    毕竟刚拿了99，有点小膨胀了，昨天两天都没刷题，在背专业课了，不过手不能生啊，每天做个一两道。

    47分钟做完了前两题，第一题做的稍微有点久，花了35分钟才ac。

## A1148 [Werewolf - Simple Version](https://pintia.cn/problem-sets/994805342720868352/problems/1038429808099098624)

```cpp
#include<iostream>
#include<vector>
using namespace std;
const int maxn = 101;
int n, werewolf[maxn],wolfliar, humanliar;

int main()
{
	cin >> n;
	for (int i = 1; i <= n; i++)
	{
		cin >> werewolf[i];
	}
	for (int i = 1; i < n; i++)
	{
		for (int j = i + 1; j <= n; j++)
		{
			wolfliar = 0, humanliar = 0;
			for (int k = 1; k <= n; k++)
			{
				if (k == i || k == j)//当狼
				{
					//说别人是狼
					if (werewolf[k] < 0 && werewolf[k] != -i && werewolf[k] != -j) wolfliar++;
					if(werewolf[k] > 0 &&( werewolf[k] == i || werewolf[k] == -j)) wolfliar++;
				}
				else
				{
					//说别人是狼
					if (werewolf[k] < 0 && werewolf[k] != -i && werewolf[k] != -j) humanliar++;
					//说狼是人
					if (werewolf[k] > 0 && (werewolf[k] == i || werewolf[k] == j)) humanliar++;
				}
			}
			if ((wolfliar + humanliar == 2) && wolfliar > 0)
			{
				cout << i << ' ' << j;
				return 0;
			}
		}
	}
	cout << "No Solution";
}
```

    这个题一开始我理解错了，我开始以为样例1第二行的那个-2意思是2号玩家指控1号玩家为狼，但实际上是1号玩家指控2号玩家为狼，就这搞反了，一开始还拿了8分......

## A1149 [Dangerous Goods Packaging](https://pintia.cn/problem-sets/994805342720868352/problems/1038429908921778176)

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
using namespace std;
unordered_map<int, vector<int>> neighbor;
int n, m, k, g1, g2, g3;

int main()
{
	cin >> n >> m;
	for (int i = 0; i < n; i++)
	{
		cin >> g1 >> g2;
		neighbor[g1].emplace_back(g2);
		neighbor[g2].emplace_back(g1);
	}
	for (int i = 0; i < m; i++)
	{
		cin >> k;
		unordered_map<int, int> map;
		for (int j = 0; j < k; j++)
		{
			cin >> g3;
			map[g3]++;
		}
		bool flag = false;
		for (auto it = map.begin(); it != map.end(); it++)
		{
			for (int g : neighbor[it->first])
			{
				if (map.count(g))
				{
					flag = true;
					break;
				}
			}
			if (flag)
			{
				cout << "No" << endl;
				break;
			}
		}
		if (!flag)cout << "Yes" << endl;
	}
}
```

    就典型的哈希表映射，13min速通。


