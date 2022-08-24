---
title: PAT-Cartesian Tree & Summit & Block Reversing
date: 2022-08-24 18:46:00 +0800
categories: [算法刷题, PAT]
tags: [堆, 模拟, 链表]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1167 [Cartesian Tree](https://pintia.cn/problem-sets/994805342720868352/problems/1478636026017230848)

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<algorithm>
using namespace std;
vector<int> inOrder(40);
int n;
const int inf = 2147483647;
struct node
{
	int v;
	node* left, * right;
	node(int _v) :v(_v), left(nullptr), right(nullptr) {}
};

node* build(int l, int r)
{
	if (l > r) return nullptr;
	int min = inf;
	int index;
	for (int i = l; i <= r; i++)
	{
		if (inOrder[i] < min)
		{
			min = inOrder[i];
			index = i;
		}
	}
	node* n = new node(min);
	n->left = build(l, index - 1);
	n->right = build(index + 1, r);
	return n;
}

int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		cin >> inOrder[i];
	}
	node* root;
	root = build(0, n - 1);

	vector<int> level;
	queue<node*> q;
	q.push(root);
	while (!q.empty())
	{
		node* top = q.front();
		q.pop();
		level.emplace_back(top->v);
		if (top->left) q.push(top->left);
		if (top->right) q.push(top->right);
	}
	for (int i = 0; i < level.size(); i++)
	{
		if (i != 0)cout << ' ';
		cout << level[i];
	}
}
```

    堆+数的构造，利用中序遍历和堆的性质建树，中序遍历中，最小的是根。用时20分36秒，まだまだ。



## A1166[Summit](https://pintia.cn/problem-sets/994805342720868352/problems/1478635919459287040)

```cpp
#include<iostream>
#include<vector>
#include<set>
#include<cstdio>
using namespace std;
const int maxn = 210;
bool direct[maxn][maxn] = { false };
int n, m, k;
bool come[maxn];
int main()
{
	cin >> n >> m;
	for (int i = 0; i < m; i++)
	{
		int a, b;
		cin >> a>>b;
		direct[a][b] = direct[b][a] = true;
	}
	cin >> k;
	for (int i = 1; i <= k; i++)
	{
		fill(come + 1, come + n + 1, false);
		int l;
		cin >> l;
		vector<int> summit(l);
		for (int j = 0; j < l; j++)
		{
			cin >> summit[j];
			come[summit[j]] = true;
		}
		bool flag = false;
		for (int j = 0; j < l - 1; j++)
		{
			for (int p = j + 1; p < l; p++)
			{
				if (direct[summit[j]][summit[p]] == false)
				{
					flag = true;
					printf("Area %d needs help.\n", i);
					break;
				}
			}
			if (flag) break;
		}
		if (flag) continue;

		
		bool flag2 = false;
		for (int j = 1; j <= n; j++)
		{
			flag = true;
			if (come[j] == false && direct[j][summit[0]] == true)//首先这个人没来还是第一个人的朋友，再判断是不是其他人的朋友
			{
				for (int p = 1; p < l; p++)
				{
					if (direct[j][summit[p]] == false)
					{
						flag = false;//这个人不符合标准，接着找
						break;
					}
				}
				if (flag)
				{
					flag2 = true;
					printf("Area %d may invite more people, such as %d.\n", i, j);
					break;
				}
			}
		}
		if (flag2 == false)
		{
			printf("Area %d is OK.\n", i);
		}
		
	}
}
```

    53分钟，拿下21/25分，速度有点忒慢了，主要是debug了半天，思路其实就是使用邻接表暴力判断，因为数据量不是很大，就200，所以再暴力也不会超时，需要注意的有以下几点：

* 如果一个人是暂定成员中的朋友，而且没来，那么这个人不一定就需要邀请过来，还得判断他是不是其他成员的朋友，所以开始我想用每个人的朋友数量来直接判断是不是OK，这样是不对的

* 有两个flag值（其实是三个，重复利用了第一个），其中在后面判断是否需要再邀请人的时候，每次大for循环都要重置flag为true，不然的话，测试点1会过不去，因为假如第一个人不符合被邀请条件，设置flag为false，那么就算后面有个人符合条件，这时候没有再设置flag为true的话，后面就打印不了了，所以每次大循环都要重置flag

## A1165 [Block Reversing](https://pintia.cn/problem-sets/994805342720868352/problems/1478635841315209216)

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
#include<cstdio>
using namespace std;

const int maxn = 100010;
vector<int> linkList;
vector<int> reverseLinkList;
unordered_map<int, int> address2v;
unordered_map<int, int> v2next;
unordered_map<int, int> v2address;
int head,n, k;
void dfs(int i)
{
	if (i + k >= linkList.size())
	{
		for (int j = i; j < linkList.size(); j++)
		{
			reverseLinkList.emplace_back(linkList[j]);
		}
		v2next[reverseLinkList.back()] = -1;
		return;
	}
	dfs(i + k);
	v2next[reverseLinkList.back()] = v2address[linkList[i]];
	for (int j = i; j < i + k; j++)
	{
		reverseLinkList.emplace_back(linkList[j]);
	}
	v2next[reverseLinkList.back()] = -1;
}
int main()
{
	cin >> head >> n >> k;
	for (int i = 0; i < n; i++)
	{
		int addr, v, next;
		cin >> addr >> v >> next;
		address2v[addr] = v;
		v2next[v] = next;
		v2address[v] = addr;
	}
	int tmp = head;
	while (tmp != -1)
	{
		linkList.emplace_back(address2v[tmp]);
		tmp = v2next[address2v[tmp]];
	}
	dfs(0);
	for (int i = 0; i < reverseLinkList.size(); i++)
	{
		if (i != reverseLinkList.size() - 1)
		{
			printf("%05d %d %05d\n", v2address[reverseLinkList[i]], reverseLinkList[i], v2next[reverseLinkList[i]]);
		}
		else printf("%05d %d %d\n", v2address[reverseLinkList[i]], reverseLinkList[i], v2next[reverseLinkList[i]]);

	}

}
```

    1小时1分钟拿下，还是太慢了，这样下去根本做不完题目的，哎。疯狂造哈希表，思路是这样的：用dfs把原序列k个一组翻转，并利用多个哈希表的映射修改next值，代码量不多，思考的地方还挺多。麻了，估计就我搁这dfs，我看别人都是直接链表翻转，麻了。


