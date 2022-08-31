---
title: PAT-N Queens Puzzle & Recommendation System & Infix Expression & Subway Map 
date: 2022-08-31 23:59:00 +0800
categories: [算法刷题, PAT]
tags: [哈希表, 排序, 二叉树, 中缀表达式, dfs, Dijkstra]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    今天的模拟测试并不是很理想，不知道是否跟状态有关。跟昨天一样，超时9min，很奇怪，昨天也差不多超时这么长时间，怎么回事呢？而且不同的时，今天3h9min32s只拿了75分，要是按3h算，最后一题没写完，那就只有59分，还没我上次PAT考的分高，麻了。接下来好好复盘下吧。

## A1128 [N Queens Puzzle](https://pintia.cn/problem-sets/994805342720868352/problems/994805348915855360)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<unordered_map>
using namespace std;

int n;
int Hash(int x, int y)
{
    return n * (x - 1) + y;
}

int main()
{
    int k;
    cin >> k;
    while (k--)
    {
        cin >> n;
        unordered_map<int, int> row;
        vector<bool> dia((n+1)*(n+1), false);
        string s;
        bool flag = true;
        for (int col = 1; col <= n; col++)
        {
            int _row;
            cin >> _row;
            if (row.count(_row))
            {
                cout << "NO" << endl;
                flag = false;
                getline(cin, s);
                break;
            }
            else
            {
                row[_row]++;
            }
            if (dia[Hash(_row, col)])
            {
                cout << "NO" << endl;
                flag = false;
                getline(cin, s);
                break;
            }
            else
            {
                int x = _row, y = col;
                while (x >= 1 && y >= 1)
                {
                    dia[Hash(x, y)]=true;
                    x--;
                    y--;
                }

                while (x < n && y < n)
                {
                    x++;
                    y++;
                    dia[Hash(x, y)] = true;
                }
                x = _row, y = col;
                while (x > 1 && y < n)
                {
                    x--;
                    y++;
                    dia[Hash(x, y)] = true;
                }
                x = _row, y = col;
                while (x < n && y > 1)
                {
                    x++;
                    y--;
                    dia[Hash(x, y)] = true;
                }
            }
        }
        if(flag) cout << "YES" << endl;
    }
}
```

    这是改了之后ac的，一开始做的时候花了40min（debug太久了）只拿了16分，最后一个测试点超时了，本来判断是否在对角线，我也用的哈希表，但会超时，而且用了超级多的内存，改成bool数组后就不超时了，用的内存也很少，看来哈希表还得谨慎使用，有那么几点需要注意：

* 自己写了个hash函数，把坐标映射成一个数，但我在调函数的时候，参数写反了，就这一个点debug了好久，麻了，一定要细心，注意参数顺序

* 还有就是，我是一遍读取一遍判断的，如果判断为NO就会提前退出，但！是！提前退出后，后面还有数字没读的，结果就把后面剩余的部分读到下一个棋盘里去了！所以提前退出的话，要把这一行剩下的数据用`getline`读掉！

* 上面两处小错debug了好久，第一题浪费的时间足够把最后一题的第一版写完了！

## A1129 [Recommendation System](https://pintia.cn/problem-sets/994805342720868352/problems/994805348471259136)

```cpp
#include<iostream>
#include<vector>
#include<cstring>
#include<unordered_map>
#include<algorithm>
using namespace std;

const int maxn = 50010;
int times[maxn];
vector<int> rec;
unordered_map<int, int> m;
bool comp(int r1, int r2)
{
    if (times[r1] != times[r2])
    {
        return times[r1] > times[r2];
    }
    else return r1 < r2;
}
int main()
{
    int n, k, r;
    memset(times, 0, sizeof(times));
    cin >> n >> k >> r;
    rec.emplace_back(r);
    times[r]++;
    m[r]++;
    for (int i = 0; i < n - 1; i++)
    {
        cin >> r;
        cout << r << ":";
        for (int j = 0; j < k && j < rec.size(); j++)
        {
            cout << ' ' << rec[j];
        }
        cout << endl;
        if (i == n - 2)break;
        if (!m.count(r))
        {
            rec.emplace_back(r);
            m[r]++;
        }
        times[r]++;
        sort(rec.begin(), rec.end(), comp);
    }
}
```

    这道题我用的暴力排序拿了18分，每次加入到数组中一个数后就排次序，果不其然，超时了，测试点三和四超时了，我改成只有当m中有r时排序，测试点四就能过了，但其他基本都没过。其实也还好了，因为用的方法简单，所以节省了不少时间。

```cpp
#include<iostream>
#include<vector>
#include<cstring>
#include<unordered_map>
#include<algorithm>
using namespace std;

const int maxn = 50010;
int times[maxn];
vector<int> rec;
unordered_map<int, int> m;
bool comp(int r1, int r2)
{
    if (times[r1] != times[r2])
    {
        return times[r1] > times[r2];
    }
    else return r1 < r2;
}
int main()
{
    int n, k, r;
    memset(times, 0, sizeof(times));
    cin >> n >> k >> r;
    rec.emplace_back(r);
    times[r]++;
    m[r]++;
    for (int i = 0; i < n - 1; i++)
    {
        cin >> r;
        cout << r << ":";
        for (int j = 0; j < k && j < rec.size(); j++)
        {
            cout << ' ' << rec[j];
        }
        cout << endl;
        if (i == n - 2)break;
        times[r]++;
        if (!m.count(r))
        {
            m[r]++;
            //insert
            int pos = rec.size() - 1;
            while (pos >= 0 && times[rec[pos]] == times[r] && rec[pos] > r)
            {
                pos--;
            }
            rec.insert(rec.begin() + pos + 1, r);

        }
        else sort(rec.begin(), rec.end(), comp);
    }
}
```

    这是我优化后的，如果是一个新的产品，那么就只是插入数组中，不做排序，如果是老产品，那就排序，这次拿了21分，依旧有一个测试点过不去。

```cpp
#include<iostream>
#include<vector>
#include<cstring>
#include<unordered_map>
#include<algorithm>
using namespace std;

const int maxn = 50010;
int times[maxn];
vector<int> rec;
bool inRec[maxn] = {false};
bool comp(int r1, int r2)
{
	if (times[r1] != times[r2])
	{
		return times[r1] > times[r2];
	}
	else return r1 < r2;
}
int main()
{
	int n, k, r;
	memset(times, 0, sizeof(times));
	cin >> n >> k >> r;
	rec.emplace_back(r);
	times[r]++;
	inRec[r] = true;
	for (int i = 0; i < n - 1; i++)
	{
		cin >> r;
		cout << r << ":";
		for (int j = 0; j < k && j < rec.size(); j++)
		{
			cout << ' ' << rec[j];
		}
		cout << endl;
		if (i == n - 2)break;
		times[r]++;
		if (inRec[r] == false)
		{
			if (rec.size() < k)
			{
				rec.emplace_back(r);
				inRec[r] = true;
			}
			else
			{
				if (times[rec.back()] < times[r] || (times[rec.back()] == times[r] && r < rec.back()))
				{
					inRec[rec.back()] = false;
					rec.pop_back();
					rec.emplace_back(r);
					inRec[r] = true;
				}
			}
		}
		sort(rec.begin(), rec.end(), comp);
	}
}
```

    这个是看了网上的博客才恍然大悟，根本没必要对整个数组都排序，只要给k个以内的数排序就行了，这样排很多次也不用怕，就看怎么更新了：

* 首先要增加一次产品的浏览次数times

* 判断该产品是否在推荐列表里
  
  * 如果不在：检查推荐列表是否已满
    
    * 如果没满，直接加入推荐列表
    
    * 如果满了，与最后一个比较，看是否能替代最后一个

* 最后把新的推荐列表根据规则排序

## A1130 [Infix Expression](https://pintia.cn/problem-sets/994805342720868352/problems/994805347921805312)

```cpp
#include<iostream>
#include<vector>
#include<cstring>
#include<unordered_map>
#include<algorithm>
using namespace std;

const int maxn = 22;
struct node
{
	string v;
	int left, right;
}tree[maxn];
bool isRoot[maxn] = { false };
int Realroot;
void travelsal(int root)
{
	if (root == -1) return;
	if (root == Realroot)
	{
		travelsal(tree[root].left);
		cout << tree[root].v;
		travelsal(tree[root].right);
	}
	else if (tree[root].left == -1 && tree[root].right != -1)
	{
		//pre
		cout << '(';
		cout << tree[root].v;
		travelsal(tree[root].right);
		cout << ')';
	}
	else//in
	{
		if(tree[root].left != -1) cout << '(';
		travelsal(tree[root].left);
		cout << tree[root].v;
		travelsal(tree[root].right);
		if (tree[root].right != -1) cout << ')';	
	}
}
int main()
{
	int n;
	cin >> n;
	for (int i = 1; i <= n; i++)
	{
		cin >>tree[i].v >> tree[i].left >> tree[i].right;
		if (tree[i].left != -1) isRoot[tree[i].left] = true;
		if (tree[i].right != -1) isRoot[tree[i].right] = true;
	}
	for (int i = 1; i <= n ; i++)
	{
		if (isRoot[i] == false)
		{
			Realroot = i;
			break;
		}
	}
	travelsal(Realroot);
}
```

    得益于昨天做的后缀表达式，今天这个中缀表达式很快就写完了，而且也是唯一一道ac的题目，需要注意的就是如果是叶子节点，不需要打括号，还有就是如果一个节点只有右子树（比如负号），这时候要特判成类后序遍历（因为不需要遍历左子树，遍历顺序为**右根**，就叫类后序遍历吧）。

## A1131 [Subway Map](https://pintia.cn/problem-sets/994805342720868352/problems/994805347523346432)

```cpp
#include<iostream>
#include<vector>
#include<cstring>
#include<set>
#include<unordered_map>
#include<algorithm>
#include<cstdio>
using namespace std;
const int inf = 0x3fffffff;
const int maxn = 101;
vector<vector<int>> neighbor(maxn);
unordered_map<int, int> station2index;
unordered_map<int, int> index2station;
unordered_map<int, set<int>> station2line;
int dis[maxn], transCnt[maxn], pre[maxn];
bool vis[maxn];
void Dijkstra(int s)
{
	fill(dis, dis + maxn, inf);
	fill(transCnt, transCnt + maxn, 0);
	fill(vis, vis + maxn, false);
	dis[s] = 0;
	transCnt[s] = 0;
	for (int i = 0; i < station2index.size(); i++)
	{
		int u = -1, min = inf;
		for (int j = 0; j < station2index.size(); j++)
		{
			if (vis[j] == false && dis[j] < min)
			{
				min = dis[j];
				u = j;
			}
		}
		if (u == -1) return;
		vis[u] = true;
		for (auto n : neighbor[u])
		{
			if (vis[n] == false)
			{
				int trCnt = transCnt[u];
				if (station2line[index2station[u]].size() > 1 && u != s)trCnt++;
				if (dis[u] + 1 < dis[n])
				{
					dis[n] = dis[u] + 1;
					pre[n] = u;
					transCnt[n] = trCnt;
				}
				if (dis[u] + 1 == dis[n] && trCnt < transCnt[n])
				{
					dis[n] = dis[u] + 1;
					pre[n] = u;
					transCnt[n] = trCnt;
				}
			}
		}
	}
}
vector<int> path;
void dfs(int start, int cur)
{
	
	if (cur == start)
	{
		path.emplace_back(cur);
		return;
	}
	dfs(start,index2station[pre[station2index[cur]]]);
	//transfer
	if(station2line[cur].size() > 1) path.emplace_back(cur);
}
int stationsLine(int s1, int s2)
{
	for (auto l1 : station2line[s1])
	{
		for (auto l2 : station2line[s2])
		{
			if (l1 == l2) return l1;
		}
	}
	return -1;
}
void printPath()
{
	for (int i = 0; i < path.size() - 1; i++)
	{
		int line = stationsLine(path[i], path[i+1]);
		printf("Take Line#%d from %04d to %04d.\n", line, path[i], path[i + 1]);
	}
}
int main()
{
	int n, m, index = 0;
	cin >> n;
	for (int i = 1; i <= n; i++)
	{
		cin >> m;
		vector<int> line(m);
		for (int j = 0; j < m; j++)
		{
			int station;
			cin >> station;
			if (!station2index.count(station))
			{
				station2index[station] = index;
				index2station[index] = station;
				index++;
			}
			station2line[station].insert(i);
			line[j] = station2index[station];
		}
		for (int j = 0; j < m - 1; j++)
		{
			neighbor[line[j]].emplace_back(line[j + 1]);
			neighbor[line[j + 1]].emplace_back(line[j]);
		}
	}
	int k;
	cin >> k;
	while (k--)
	{
		int start, end;
		cin >> start >> end;
		Dijkstra(station2index[start]);
		cout << dis[station2index[end]]<<endl;
		path.clear();
		dfs(start, end);
		path.emplace_back(end);
		printPath();
	}
}

```

    这些代码写了一个半小时，也没写完，最后超时9min才写完，就这也就拿了16/30分，麻了。关键是debug了很久，光算最短路的时候就debug了一会儿，发现n写成了u，哎。还有就是各种哈希映射太多了，有的搞混了，要么中断，要么答案有问题。我看网上是有用dfs写的，今天不早了，明天看看。
