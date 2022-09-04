---
title: PAT-2022年秋季考试-Balloon Popping & The Second Run of Quicksort & Leader of the Opinion Leaders & Pseudo-completeness
date: 2022-09-04 22:42:00 +0800
categories: [算法刷题, PAT]
tags: [双指针, 排序, 模拟, 二叉树的遍历, 完全二叉树]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    这次考试过程就像过山车，看到题目就有点没底，从59分一点一点蚕食，最终拿到了99分，比以往的模拟考的都要好，属于是超常发挥了哈哈。虽然没能拿满分有点小遗憾（现在我也知道是哪没过了），但99分也蛮不错了，心中有点小窃喜。

## Balloon Popping

```cpp
#include<iostream>
#include<vector>
using namespace std;
const int maxl = 2000001;
const int maxn = 100001;
int n, h, co[maxn],predix[maxn],maxb = 1, minpos;
int l, r;
int main()
{
	cin >> n >> h;
	for (int i = 0; i < n; i++)
	{
		cin >> co[i];
	}
	if (n == 1)
	{
		cout << co[0] - h << ' ' << 1;
		return 0;
	}
	l = 0, r = 1;
	while (r < n && l < r)
	{
		while (r < n && co[r] - co[l] <= h )
		{
			r++;
		}
		if (co[r] - co[l] > h)
		{
			if (r - l > maxb)
			{
				maxb = r - l;
				minpos = co[r-1] - h;
			}
		}
		else if (r == n && co[r] - co[l] <= h)
		{
			
			if (r - l + 1> maxb)
			{
				maxb = r - l + 1;
				minpos = co[r - 1] - h;
			}
		}
		l++;
	}
	cout << minpos << ' ' << maxb;
}
```

    这道题一开始我用的暴力算法，直接挨个查，只过了第一个点，12分，剩下的不是答案错误就是超时，后来我在想可以倒过来再查一遍，果然，又拿了1分。最后写完后三道题后，又回来想办法，最后剩十几分钟的时候，想到了双指针法，赶紧实现，最后拿了19/20分，差一个测试点没过，我晚上洗澡的时候突然就想到了，是只有一个气球的情况没考虑，因为我的r是从1开始算的，虽然我的最大气球数量默认是1，但距离就得是第一个坐标减去身高了。

## The Second Run of Quicksort

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;
const int maxn = 100010;
vector<int> sequence;
int maxL, minR, cnt, k, n, tmp;
bool bigger[maxn];
bool pivot[maxn];
int main()
{
	cin >> k;
	while (k--)
	{
		scanf("%d", &n);
		//cin >> n;
		sequence.clear();
		for (int i = 0; i < n; i++)
		{
			scanf("%d", &tmp);
			//cin >> tmp;
			sequence.emplace_back(tmp);
		}
		fill(bigger, bigger + n, false);
		fill(pivot, pivot + n, false);
		maxL = -1, minR = 0x3fffffff;
		cnt = 0;
		for (int i = 0; i < n; i++)
		{
			if (sequence[i] > maxL)
			{
				bigger[i] = true;
				maxL = sequence[i];
			}
		}
		for (int i = n - 1; i >= 0; i--)
		{
			if (bigger[i] && sequence[i] < minR)
			{
				cnt++;
				pivot[i] = true;
				if (cnt == 3) break;
			}
			if (sequence[i] < minR)minR = sequence[i];
		}
		if (cnt >= 3) cout << "Yes" << endl;
		else if (cnt == 2 && (pivot[0] || pivot[n - 1]))cout << "Yes" << endl;
		else cout << "No" << endl;

	}
}
```

    这道题我一看是快排，立马放弃，因为我完全忘记快排咋排了，考前我还给室友说我快排没看来着，真是不会啥考啥，本来我都做好只考七十多分的准备了。做完最后一题后就又回来看，往下一拉，发现下面有提示，就是对四个样例的说明，好家伙，直接被我找到规律了，就去查序列中满足大于左边所有数，小于右边所有数的数就行了，中间debug了一会，可算是找到规律了。如果一个序列有三个这样的数就可以是快排，如果有两个，其中一个在0号位或者末尾的话也行。

## Leader of the Opinion Leaders

```cpp
#include<iostream>
#include<vector>
using namespace std;
const int maxn = 10010;

struct node
{
	int ol = -1, follows = 0;
};
vector<node> user(maxn);
vector<vector<int>> fan(maxn);
vector<int> ans;
int n, t, follow, f, maxc = 0;
int main()
{
	cin >> n >> t;
	for (int i = 1; i <= n; i++)
	{
		cin >> follow;
		user[i].follows = follow;
		for (int j = 0; j < follow; j++)
		{
			cin >> f;
			fan[f].emplace_back(i);
		}
	}
	for (int i = 1; i <= n; i++)
	{
		if (fan[i].size() / user[i].follows >= t)
		{
			user[i].ol = 1;
		}
	}
	for (int i = 1; i <= n; i++)
	{
		if (user[i].ol == 1)
		{
			int cnt = 0;
			for (int fa : fan[i])
			{
				if (user[fa].ol == 1) cnt++;
			}
			if (cnt > maxc)
			{
				maxc = cnt;
				ans.clear();
				ans.emplace_back(i);
			}
			else if (cnt == maxc)
			{
				ans.emplace_back(i);
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

    这道题是最简单的一道，虽然做的时候挺着第一道做了一半、第二题没想法的压力写得不够好，但依旧是直接ac了。题库里有类似的题目嘻嘻。

## Pseudo-completeness

```cpp
#include<iostream>
#include<vector>
#include<queue>
using namespace std;
const int maxn = 2010;
struct node
{
	int v, level;
	node* left = nullptr, * right = nullptr;
	node(int _v, int _l) : v(_v), level(_l) {}
};
vector<int> in, pre, post;
int maxl = 0;
node* build(int inL, int inR, int preL, int preR, int level)
{
	if (inL > inR) return nullptr;
	node* n = new node(pre[preL], level);
	if (level > maxl) maxl = level;
	int index;
	for (int i = inL; i <= inR; i++)
	{
		if (in[i] == pre[preL]) index = i;
	}
	n->left = build(inL, index - 1, preL + 1, preL + index - inL, level + 1);
	n->right = build(index + 1, inR, preL + index - inL + 1, preR, level + 1);
	return n;
}
bool f1 = true, f2 = true, f3 = true;
void levelTravelsal(node* root)
{
	queue<node*> q;
	q.push(root);
	bool iscom = false;
	int leaflevel = -1;
	while (!q.empty())
	{
		node* top = q.front();
		q.pop();
		if (!top->left && !top->right)
		{
			if (leaflevel == -1)leaflevel = top->level;
			else if (leaflevel != top->level)
			{
				f1 = false;
			}
			if (maxl - 1 > top->level) f3 = false;
		}

		if ((top->left && !top->right) || (!top->left && top->right))
		{
			f1 = false;
			if (top->level < maxl - 1) f3 = false;
		}
		if (top->left && iscom) f2 = false;
		if (top->left)q.push(top->left);
		else iscom = true;
		if (top->right && iscom) f2 = false;
		if (top->right) q.push(top->right);
		else iscom = true;
	}
}
void postTravelsal(node* root)
{
	if (root == nullptr) return;
	postTravelsal(root->left);
	postTravelsal(root->right);
	post.emplace_back(root->v);
}
int main()
{
	int n, tmp;
	cin >> n;
	for (int i = 0; i < n; i++)
	{
		cin >> tmp;
		in.emplace_back(tmp);
	}
	for (int i = 0; i < n; i++)
	{
		cin >> tmp;
		pre.emplace_back(tmp);
	}
	node* root;
	root = build(0, n - 1, 0, n - 1, 0);
	levelTravelsal(root);
	if (f1) cout << 1 << endl;
	else if (f2) cout << 2 << endl;
	else if (f3)cout << 3 << endl;
	else cout << 0 << endl;
	postTravelsal(root);
	for (int i = 0; i < n; i++)
	{
		if (i != 0) cout << ' ';
		cout << post[i];
	}

}
```

    虽然这道题代码量不小，但这道题并不难，因为里面涉及的知识点在暑假都遇到过，首先是建树，太典了，然后就是完美二叉树的判断、完全二叉树的判断（这个我还专门复习过两遍），就是这个新加的某某树的判断没见过，不过也不难，就在遍历的时候判断就行了，需要注意的是，是完美二叉树的条件有两个，一个就是非叶子节点必有两个孩子，还有一个就是叶子节点在同一层，所以在判断第三种树的时候，这两种条件也要想到，我就是一开始没想到这两种情况，就拿了21分，后来想起来了，就拿满了。
