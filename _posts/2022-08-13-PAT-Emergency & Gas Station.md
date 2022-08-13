---
title: PAT-Emergency & Gas Station
date: 2022-08-13 19:51:00 +0800
categories: [算法刷题, PAT]
tags: [Djkstra, Bellman-Ford]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1003 [Emergency](https://pintia.cn/problem-sets/994805342720868352/problems/994805523835109376)

```cpp
#include<iostream>
#include<vector>
#include<queue>
using namespace std;
struct node
{
	int v, dis;//目标节点编号，距离
	node(int _v, int _dis) :v(_v), dis(_dis) {}
};
const int maxn = 500;
const int inf = 0x3fffffff;
vector<vector<node>> neighbor(maxn);//每个节点的邻接表
bool isVisited[maxn] = {false};//访问数组
//记录源节点到每个节点的最短距离,到达某个节点的最大队伍数量，到达某个节点的最短路径数量
int dis[maxn], teamNum[maxn], pathNum[maxn];
int team[maxn];//记录每个节点的队伍数量
int n, m, c1, c2;//节点数、路数、c1、c2

//迪杰斯特拉算法
void Djkstra()
{
	fill(dis, dis + n, inf);//初始化到达所有结点的最短距离为极大值
	dis[c1] = 0;//先把出发点的距离设置为0
	teamNum[c1] = team[c1];
	pathNum[c1] = 1;
	//接下来要开始遍历n次（攻占n个城市）
	for (int i = 0; i < n; i++)
	{
		//查未访问的节点中，dis最短的（也就是从出发点到目标点距离最短的）
		int u = -1, min = inf;//目标点，最短距离
		for (int j = 0; j < n; j++)
		{
			if (!isVisited[j] && dis[j] < min)
			{
				u = j;
				min = dis[j];
			}
		}
		//先判断是否存在这么一个点
		if (u == -1) return;//说明不是连通图
		isVisited[u] = true;//标记该节点已访问过
		//开始查u的每一个邻居，更新每个邻居的dis值
		for (auto n : neighbor[u])
		{
			//如果以u为中介节点，到达n节点，距离小于原来的dis
			if (dis[u] + n.dis < dis[n.v])
			{
				dis[n.v] = dis[u] + n.dis;//更新该节点的dis值
				pathNum[n.v] = pathNum[u];//到该节点的最短路径更新为到达u的路径总数
				teamNum[n.v] = teamNum[u] + team[n.v];//更新救援队伍人数
			}
			else if (dis[u] + n.dis == dis[n.v])
			{
				pathNum[n.v] += pathNum[u];//到该节点的最短路径+到达u的路径总数
				if (teamNum[u] + team[n.v] > teamNum[n.v])//如果走这条路，救援人员更多
				{
					teamNum[n.v] = teamNum[u] + team[n.v];//更新救援队伍人数
				}
			}
		}
	}
}

int main()
{
	cin >> n >> m >> c1 >> c2;
	//储存每个节点救援队伍数量
	for (int i = 0; i < n; i++)
	{
		cin>>team[i];
	}
	//记录每条路
	for (int i = 0; i < m; i++)
	{
		int _c1, _c2, _dis;
		cin >> _c1 >> _c2 >> _dis;
		neighbor[_c1].emplace_back(node(_c2, _dis));
		neighbor[_c2].emplace_back(node(_c1, _dis));
	}
	Djkstra();
	cout << pathNum[c2] << ' ' << teamNum[c2];
	
}
```

    **Dijkstra**算法：

    虽然之前做过，但再一次做仍然没能做出来，甚至还是先看了一遍算法笔记的迪杰斯特拉算法详解，也没做出来，还是得看源码。

* 迪杰斯特拉算法是计算并记录出发点到每一个点的最短距离，感觉有点像贪心+动态规划

* 计算前，需要初始化的也就是出发点的数据

* 在做最短路径数量更新时，要记得更新为中介节点路径的数量，而不是归1

* 不能忘记将遍历过的节点设置为true

```cpp
#include<iostream>
#include<vector>
#include<set>
using namespace std;
struct node
{
	int v, dis;//目标节点编号，距离
	node(int _v, int _dis) :v(_v), dis(_dis) {}
};
const int maxn = 500;
const int inf = 0x3fffffff;
vector<vector<node>> neighbor(maxn);//每个节点的邻接表
//记录源节点到每个节点的最短距离,到达某个节点的最大队伍数量，到达某个节点的最短路径数量
int dis[maxn], teamNum[maxn], pathNum[maxn];
int team[maxn];//记录每个节点的队伍数量
int n, m, c1, c2;//节点数、路数、c1、c2
vector<set<int>>pre(maxn);//记录每个节点的前驱节点
//贝尔曼福德算法
void Bellman()
{
	fill(dis, dis + n, inf);//初始化单源最短路径为极大值
	dis[c1] = 0;//到出发点的最短路径为0
	pathNum[c1] = 1;//初始化到出发点的路有一条
	teamNum[c1] = team[c1];//初始化到达c1时的救援队人数
	//遍历n-1次（因为一共n个节点，形成的最短路径树不会超过n层，也就是说最多n-1层
	for (int i = 0; i < n - 1; i++)
	{
		//每一次循环都要遍历每一条边
		for (int j = 0; j < n; j++)
		{
			for (auto n : neighbor[j])
			{
				//如果经过j节点到n节点的距离小于之前记录的dis
				if (dis[j] + n.dis < dis[n.v])
				{
					dis[n.v] = dis[j] + n.dis;//更新单源最短路(松弛操作)
					teamNum[n.v] = teamNum[j] + team[n.v];//更新救援队人数
					pathNum[n.v] = pathNum[j];//更新最短路条数
					pre[n.v].clear();//清空之前的前缀节点
					pre[n.v].insert(j);//加入这个中介节点
				}
				else if (dis[j] + n.dis == dis[n.v])//如果相等
				{
					pre[n.v].insert(j);//加入这个中介节点
					if (teamNum[j] + team[n.v] > teamNum[n.v])//如果这条路上的救援队员人数更多
					{
						teamNum[n.v] = teamNum[j] + team[n.v];//更新救援队人数
					}
					//重新计算最短路径条数
					pathNum[n.v] = 0;
					for (auto p : pre[n.v])
					{
						pathNum[n.v] += pathNum[p];
					}
				}
			}
		}
	}
}

int main()
{
	cin >> n >> m >> c1 >> c2;
	//储存每个节点救援队伍数量
	for (int i = 0; i < n; i++)
	{
		cin>>team[i];
	}
	//记录每条路
	for (int i = 0; i < m; i++)
	{
		int _c1, _c2, _dis;
		cin >> _c1 >> _c2 >> _dis;
		neighbor[_c1].emplace_back(node(_c2, _dis));
		neighbor[_c2].emplace_back(node(_c1, _dis));
	}
	Bellman();
	cout << pathNum[c2] << ' ' << teamNum[c2];
	
}
```

    **Bellman-Ford**算法：

    这个算法有点暴力，需要一直遍历，但就是因为暴力，可以解决负权的问题，需要注意的点：

* 忘记初始化出发点队伍人员数量了，导致计算结果有误

* 因为bellman算法没有访问数组（isVisited[]），所以有些中介节点是要多次遍历到的，这时候就要使用set记录前缀节点编号，**而且**，每当遇到相同长度的路径，都要重新根据前缀节点来计算一次路径数目，哪怕是遇到了相同的中介节点，因为又一次遍历的中介节点可能其路径数和人员数都有所变化了。

## A1072 [Gas Station](https://pintia.cn/problem-sets/994805342720868352/problems/994805396953219072)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<cstdio>
using namespace std;
int n, m, k, d;//居民房数目、加油站候选地数目、路数、最大服务范围
const int maxn = 1020;
const int inf = 0xfffffff;
struct node
{
	int v, dis;//邻接表中，目标节点编号，到目标节点距离
	node(int _v, int _dis) :v(_v), dis(_dis) {}
};
double minium[11];//记录加油站到最近一个居民楼的距离
double average[11];//记录加油站到每个居民区的平均距离
bool isVisited[maxn];
vector<vector<node>> neighbor(maxn);//其中1~n为居民楼，n+1~n+m是加油站
int dis[maxn];//记录某个加油站到各个居民楼的最短距离
int minDis, sumDis;//记录每个加油站距离居民楼的最近距离、加油站距离所有居民楼的距离之和
void Dijkstra(int s)
{
	//初始化数据
	fill(isVisited + 1, isVisited + n + m + 1, false);
	fill(dis + 1, dis + n + m + 1, inf);
	dis[s] = 0;
	minDis = inf;
	for (int i = 0; i < n + m; i++)//要算出到n + m个节点的距离（包括自己和其他加油站）
	{
		int u = -1, min = inf;
		//从所有节点中遍历没有访问过的，dis值最小的节点
		for (int j = 1; j <= n + m; j++)
		{
			if (isVisited[j] == false && dis[j] < min)
			{
				min = dis[j];
				u = j;
			}
		}
		//判断是否有没访问的dis值最小的点
		if (u == -1) return;//要么遍历完了，要么不连通了
		isVisited[u] = true;//设置为访问过
		//遍历u能到的所有节点，看是否需要更新最短距离
		for (node& nd : neighbor[u])
		{
			if (dis[u] + nd.dis < dis[nd.v])
			{
				dis[nd.v] = dis[u] + nd.dis;//更新距离
				//判断是否要更新最近的距离(必须是居民楼，不能是和其他加油站的距离)
				if (dis[nd.v] < minDis && nd.v <= n) minDis = dis[nd.v];
			}
		}
	}
	//判断是否有居民楼超出服务区
	sumDis = 0;
	for (int i = 1; i <= n; i++)
	{
		if (dis[i] > d)
		{
			minDis = inf;
			break;
		}
		else
		{
			sumDis += dis[i];
		}
	}
}

int main()
{
	cin >> n >> m >> k >> d;
	for (int i = 0; i < k; i++)
	{
		string _s1, _s2;//节点编号
		int _p1, _p2, _d;//节点编号、两点距离
		cin >> _s1 >> _s2 >> _d;
		//判断是否为加油站，其中1~n为居民楼，n+1~n+m是加油站
		if (_s1[0] == 'G')
		{
			_p1 = n + stoi(_s1.substr(1));
		}
		else _p1 = stoi(_s1);

		if (_s2[0] == 'G')
		{
			_p2 = n + stoi(_s2.substr(1));
		}
		else _p2 = stoi(_s2);
		neighbor[_p1].emplace_back(node(_p2, _d));
		neighbor[_p2].emplace_back(node(_p1, _d));
	}
	//遍历每一个加油站
	vector<int> minD(11);//储存每一个加油站距离居民楼的最近距离
	vector<int> sumD(11);//储存每一个加油站到各个居民楼的距离之和
	for (int i = 1; i <= m; i++)
	{
		Dijkstra(n + i);
		if (minDis == inf)//说明不在服务区
		{
			minD[i] = -1;
		}
		else
		{
			minD[i] = minDis;
			sumD[i] = sumDis;
		}
	}
	//比较并打印结果
	int maxDis = -1;//用来记录当中最大的最短距离
	int sum = 0;//记录对应的sum
	int station = -1;//记录要建在哪个站
	for (int i = 1; i <= m; i++)
	{
		if (minD[i] > maxDis)
		{
			maxDis = minD[i];//更新maxDis
			sum = sumD[i];
			station = i;
		}
		else if (minD[i] == maxDis && sumD[i] < sum)
		{
			maxDis = minD[i];//更新maxDis
			sum = sumD[i];
			station = i;
		}
	}
	if (station == -1) printf("No Solution");
	else printf("G%d\n%.1lf %.1lf", station, (double)minD[station], (double)sumD[station] / (double)n);
}
```

    这道题是迪杰斯特拉算法，debug了好久......

* 要注意，虽然加油站可以不建，但可以当隐形节点去经过

* 跑的时候有中断，问题出在`_p1 = n + stoi(_s1.substr(1));`这一行，一开始我寻思就一行代码，不加花括号了，然后就中断了，加了花括号括起来，就没中断了

* 有一次跑的时候，发现记录的minDis（也就是每个加油站距离居民楼最近的距离）不是inf（超出服务区）就是0，debug了下，发现是在遍历u能到的所有节点，看是否需要更新最短距离处，`if (dis[u] + nd.dis < dis[nd.v])`这行代码，写成了`if (dis[u] + dis[nd.dis] < dis[nd.v])`

* 最后一处，就是发现最大的距离求对了，但平均数不对，debug了半天，发现是某一个dis算错了，于是开始查，查着查着发现怎么2号节点成了2号节点自己的邻居了？于是查输入，发现把`_s2`打成`_s1`了，麻了......最大距离都能求对是我没想到的......

* 太马虎了，注意力可得提高点




