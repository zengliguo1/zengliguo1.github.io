---
title: PAT-The Missing Number & Chain the Ropes & Eulerian Path & ZigZagging on a Tree
date: 2022-09-17 15:30:00 +0800
categories: [算法刷题, PAT]
tags: [双指针, 哈夫曼树, 欧拉图, 二叉树的遍历, 树的构造]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    今天又做了一套模拟题，前三道题花了不到50分钟，但第三道题只拿了19/25分，没继续想，继续往下做了，花了二十多分钟就把最后一题写完了，1h10min的分数为94，接着回过头去想第3题，又写了20min通过了。

    本日战绩：1h30min 100分。

    后天就要考试了，明天就不刷题了，就复习复习做过的题目吧。

## A1144 [The Missing Number](https://pintia.cn/problem-sets/994805342720868352/problems/994805343463260160)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<cstdio>
using namespace std;
int n, tmp;
vector<int> l;
int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		scanf("%d", &tmp);
		if (tmp > 0) l.emplace_back(tmp);
	}
	sort(l.begin(), l.end());
	if (l.size() == 0) cout << 1;
	else if (l[0] > 1) cout << 1;
	else
	{
		bool flag = true;
		for (int i = 0; i < l.size() - 1; i++)
		{
			if (l[i + 1] > l[i] + 1)
			{
				cout << l[i] + 1;
				flag = false;
				break;
			}
		}
		if (flag) cout << l.back() + 1;
	}
}
```

    算双指针吗？有点像又不太像，需要注意一种情况就是，整个序列是连续的，那么缺少的最小值就是最后一个值+1。

## A1125 [Chain the Ropes](https://pintia.cn/problem-sets/994805342720868352/problems/994805350316752896)

```cpp
#include<iostream>
#include<queue>
using namespace std;
int n;
double tmp;
priority_queue<double, vector<double>, greater<double>> q;
int main()
{
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		cin >> tmp;
		q.push(tmp);
	}
	while (q.size() > 1)
	{
		double t1 = q.top();
		q.pop();
		double t2 = q.top();
		q.pop();
		q.push((t1 + t2) / 2);
	}
	int l = (int)q.top();
	cout << l;
}
```

    简单的哈夫曼树题，就是带权路径最小的树，直接用小顶堆即可。

## A1126 [Eulerian Path](https://pintia.cn/problem-sets/994805342720868352/problems/994805349851185152)

```cpp
#include<iostream>
#include<vector>
using namespace std;
const int maxn = 501;
int n, m, u, v, degree[maxn], oddcnt, notconnect, connectcnt;
bool isVis[maxn];
vector<vector<int>> neighbor(maxn);
void dfs(int s)
{
	if (s > n) return;
	isVis[s] = true;
	connectcnt++;
	for (int n : neighbor[s])
	{
		if (!isVis[n])
		{
			dfs(n);
		}
	}
}
int main()
{
	cin >> n >> m;
	for (int i = 0; i < m; i++)
	{
		cin >> u >> v;
		degree[u]++;
		degree[v]++;
		neighbor[u].emplace_back(v);
		neighbor[v].emplace_back(u);
	}
	for (int i = 1; i <= n; i++)
	{
		if (i != 1) cout << ' ';
		cout << degree[i];
		if (degree[i] % 2 == 1) oddcnt++;
	}
	dfs(1);
	if(connectcnt != n) notconnect = 1;
	cout << endl;
	if (notconnect == 0)
	{
		if (oddcnt == 0) cout << "Eulerian";
		else if (oddcnt == 2) cout << "Semi-Eulerian";
		else cout << "Non-Eulerian";
	}
	else cout << "Non-Eulerian";
}
```

    判断欧拉环路和欧拉路，正好了解下欧拉图的判别：连通图中，

* 所有节点的度为偶数，那么就有欧拉环路

* 有且仅有两个节点的度为奇数，那么就有欧拉路

    这道题一开始没拿满，是因为没有判别是否为连通图，dfs一遍就能知道了，后面还有一个点过不去，就是因为，当只有一个节点的时候，我给的结果不是欧拉图，但答案是欧拉环路，蚌。

## A1127 [ZigZagging on a Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805349394006016)

```cpp
#include<iostream>
#include<queue>
#include<vector>
using namespace std;
const int maxn = 31;
struct node
{
	int v, l;
	node* left, * right;
	node(int _v) : v(_v), left(nullptr), right(nullptr) {}
};
int in[maxn], post[maxn], n;
vector<vector<int>> levelOrder(maxn);
node* build(int inL, int inR, int postL, int postR)
{
	if (inL > inR) return nullptr;
	node* n = new node(post[postR]);
	int index;
	for (int i = inL; i <= inR; i++)
	{
		if (in[i] == post[postR])
		{
			index = i;
			break;
		}
	}
	n->left = build(inL, index - 1, postL, postL + index - 1 - inL);
	n->right = build(index + 1, inR, postR - inR + index, postR - 1);
	return n;
}
void travelsal(node* root, int level)
{
	if (root == nullptr) return;
	root->l = level;
	travelsal(root->left, level + 1);
	travelsal(root->right, level + 1);
}
int main()
{
	cin >> n;
	for (int i = 0; i < n; i++) cin >> in[i];
	for (int i = 0; i < n; i++) cin >> post[i];
	node* root = build(0, n - 1, 0, n - 1);
	travelsal(root, 1);
	queue<node*> q;
	q.push(root);
	while (!q.empty())
	{
		node* n = q.front();
		q.pop();
		levelOrder[n->l].emplace_back(n->v);
		if (n->left)q.push(n->left);
		if (n->right)q.push(n->right);
	}
	vector<int> ans;
	for (int i = 1; i <= 30 ; i++)
	{
		if (levelOrder[i].size() == 0) break;
		if (i % 2 == 1)
		{
			for (int j = levelOrder[i].size() - 1; j >= 0; j--)
			{
				ans.emplace_back(levelOrder[i][j]);
			}
		}
		else
		{
			for (int j = 0; j < levelOrder[i].size(); j++)
			{
				ans.emplace_back(levelOrder[i][j]);
			}
		}
	}
	for (int i = 0; i < ans.size(); i++)
	{
		if (i != 0) cout << ' ';
		cout << ans[i];
	}
}

```

    就是一简单的建树、遍历，虽然它说不简单，但实际上就多一步存储节点罢了，仍然很简单，昨天那道树的构造确实不简单，昨天那个写了一个小时，今天这个就20分钟。


