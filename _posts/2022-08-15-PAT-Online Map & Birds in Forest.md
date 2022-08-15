---
title: PAT-Online Map & Birds in Forest
date: 2022-08-15 21:39:00 +0800
categories: [算法刷题, PAT]
tags: [Djkstra, 并查集]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1111 [Online Map](https://pintia.cn/problem-sets/994805342720868352/problems/994805358663417856)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;
struct node
{
    int v,len, time;
    node(int _v, int _len, int _time) : v(_v), len(_len), time(_time) {}
};
int n, m;
const int maxv = 510;
const int inf = 0xfffffff;
vector<vector<node>> neighbor(maxv);
int dis[maxv], preDis[maxv];
int t[maxv],preT[maxv], numPath[maxv];
bool vis[maxv] = {false};
void DijkstraDis(int s)
{
    fill(dis, dis + maxv, inf);
    fill(t, t + maxv, inf);
    dis[s] = 0;
    t[s] = 0;
    for (int i = 0; i < n; i++)
    {
        int u = -1, min = inf;
        for (int j = 0;j < n; j++)
        {
            if (vis[j] == false && dis[j] < min)
            {
                u = j;
                min = dis[j];
            }
        }
        if (u == -1) return;
        vis[u] = true;
        for (node n : neighbor[u])
        {
            if (vis[n.v] == false)
            {
                if (dis[u] + n.len < dis[n.v])
                {
                    dis[n.v] = dis[u] + n.len;
                    t[n.v] = t[u] + n.time;
                    preDis[n.v] = u;
                }
                else if (dis[u] + n.len == dis[n.v] && t[u] + n.time < t[n.v])
                {
                    t[n.v] = t[u] + n.time;
                    preDis[n.v] = u;
                }
            }

        }

    }
}

void DijkstraT(int s)
{
    fill(t, t + maxv, inf);
    fill(vis, vis + maxv, false);
    fill(numPath, numPath + maxv, inf);
    numPath[s] = 0;
    t[s] = 0;
    for (int i = 0; i < n; i++)
    {
        int u = -1, min = inf;
        for (int j = 0; j < n; j++)
        {
            if (vis[j] == false && t[j] < min)
            {
                u = j;
                min = t[j];
            }
        }
        if (u == -1) return;
        vis[u] = true;
        for (node n : neighbor[u])
        {
            if (vis[n.v] == false)
            {
                if (t[u] + n.time < t[n.v])
                {
                    t[n.v] = t[u] + n.time;
                    preT[n.v] = u;
                    numPath[n.v] = numPath[u] + 1;
                }
                else if (t[u] + n.time == t[n.v] && numPath[u] + 1 < numPath[n.v])
                {
                    numPath[n.v] = numPath[u] + 1;
                    preT[n.v] = u;
                }
            }
        }
    }
}
bool isSame(int s, int e)
{
    while (preDis[e] == preT[e] && e != s)
    {
        e = preDis[e];
    }
    if (e == s) return true;
    else return false;
}
void dfsDis(int start, int end)
{
    if (end == start)
    {
        cout << start;
        return;
    }
    dfsDis(start, preDis[end]);
    cout << " -> " << end;
}
void dfsT(int start, int end)
{
    if (end == start)
    {
        cout << start;
        return;
    }
    dfsDis(start, preT[end]);
    cout << " -> " << end;
}
void dfs(int pre[], vector<int>& path, int start, int end)
{
    if (end == start)
    {
        path.emplace_back(end);
        return;
    }
    dfs(pre, path, start, pre[end]);
    path.emplace_back(end);
}
void printPath(vector<int>& path)
{
    for (int i = 0; i < path.size(); i++) {
        if (i != 0) cout << " -> ";
        cout << path[i];
    }
    cout << endl;
}
int main()
{
    //freopen("input.txt", "r", stdin);
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int v1, v2, isOne, len, time;
        cin >> v1 >> v2 >> isOne >> len >> time;
        neighbor[v1].emplace_back(node(v2, len, time));
        if (isOne == 0)
        {
            neighbor[v2].emplace_back(node(v1, len, time));
        }

    }
    int start, end;
    cin >> start >> end;
    DijkstraDis(start);
    DijkstraT(start);
    vector<int> pathD;
    vector<int> pathT;
    dfs(preDis, pathD, start, end);
    dfs(preT, pathT, start, end);
    /*if (isSame(start, end))
    {
        cout << "Distance = " << dis[end] << "; Time = " << t[end] << ": ";
        dfsDis(start, end);

        cout << endl;
    }
    else
    {
        cout << "Distance = " << dis[end] << ": ";
        dfsDis(start, end);
        cout << endl;
        cout << "Time = " << t[end] << ": ";
        dfsT(start, end);
        cout << endl;
    }*/
    if (pathD == pathT)
    {
        cout << "Distance = " << dis[end] << "; Time = " << t[end] << ": ";
        printPath(pathD);
    }
    else
    {
        cout << "Distance = " << dis[end] << ": ";
        printPath(pathD);
        cout << "Time = " << t[end] << ": ";
        printPath(pathT);
    }
}
```

    我自己写的初版代码，测试点2和4过不去，只有20分，花了一小时写的，真考到这种题，估计也不继续找bug了，20就20吧。我去网上搜了好久，找到一个跟我思路几乎一模一样，只有一点不一样，我直接dfs打印结果，他是先dfs把结果存到数组，再打印结果，然后我改成这样，测试点2和4就过了......很无语，那说明我那个直接打印结果写的有问题，但我已经检查烂了，也检查不出来啥问题，哎，看不到样例就是这样的，极度痛苦，而且牛客上只有69道题，这道题根本就没有，所以也没发去牛客看样例了。总之，做了1h，找了1h的bug，最后其实也没找到bug，算是积累了个经验，凡是需要打印路径啥的，一定，千万要先存到数组里，再打印，这样好确保万无一失地拿到分数。接下来说说注意的点吧：

* 目前先不写注释了，节省时间来刷题

* 注意在遍历邻接表时，加一个判断，如果没访问过也就是visited数组为false再去判断，不加也行，加上无非是节省一点点时间

* 使用freopen来节省复制粘贴样例的时间，提交时注释掉即可

* 注意遇到重复性代码时，谨慎复制粘贴，很容易出现某个变量忘记改的问题

    **麻了！** 就在我码完上面这些字后，找到问题了！！！就在代码里（我就不改了），我想着把我自己写的直接dfs打印给注释掉吧，然后发给发小看看我这俩小时的痛苦代码，结果注释时，忘记注释`dfsT`这个函数了，紧接着出现了**红色波浪线** ，我一看，好家伙，原来是在`dfsT`的函数里，用了`dfsDis`的函数来递归（源代码还在上面，我没改，可以看下，笑死了），就这还能过仨样例，太神奇了。

## A1118 [Birds in Forest](https://pintia.cn/problem-sets/994805342720868352/problems/994805354108403712)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<set>
using namespace std;

const int maxn = 10010;
vector<int> parent(maxn), Rank(maxn);
int n;
set<int> birds, isRoot;
void Init()
{
	for (int i = 0; i <maxn; i++)
	{
		parent[i] = i;
	}
	fill(Rank.begin(), Rank.begin() + maxn, 0);
}

int Find(int b)
{
	int a = b;
	while (b != parent[b])
	{
		b = parent[b];
	}
	
	while (a != parent[a])
	{
		int z = a;
		a = parent[a];
		parent[z] = b;
	}
	return b;
}

void Union(int a, int b)
{
	a = Find(a);
	b = Find(b);
	if (a == b) return;
	if (Rank[a] > Rank[b])
	{
		parent[b] = a;
	}
	else if (Rank[b] > Rank[a])
	{
		parent[a] = b;
	}
	else
	{
		parent[b] = a;
		Rank[a]++;
	}
}

int main()
{
	//freopen("input.txt", "r", stdin);
	cin >> n;
	Init();
	for (int i = 0; i < n; i++)
	{
		int k, b0;
		cin >> k >> b0;
		birds.insert(b0);
		for (int j = 1; j < k; j++)
		{
			int bj;
			cin >> bj;
			Union(b0, bj);
			birds.insert(bj);
		}
	}
	for (int b : birds)
	{
		isRoot.insert(Find(b));
	}
	cout << isRoot.size() << ' ' << birds.size() << endl;
	int q;
	cin >> q;
	while (q--)
	{
		int b1, b2;
		cin >> b1 >> b2;
		if (Find(b1) == Find(b2))
		{
			cout << "Yes" << endl;
		}
		else
		{
			cout << "No" << endl;
		}
	}
}
```

    简单的并查集，35分钟拿下，其实25分钟就足够了，中间发生了两次段错误。第一次是因为在读取`b0`之前就插入`set`里了，此时b0是个负数，就引发了中断；第二次是因为题目最大数量是1万个，我初始化就给了1千个，少了个0，就出现了段错误。其实还有一处小错误，虽然不会影响答案，就是路径压缩时，没压好，应该是`parent[z]=b`，写成了`parent[a]=b`，不过不会影响答案，只是会导致第一个值没有压缩路径。还有一处可以优化的，在`Union`时，如果两个节点的根节点一样，直接返回就行了，不用再进行下去了。
