---
title: PAT-Public Bike Management & Pop Sequence & Mice and Rice
date: 2022-08-27 22:43:00 +0800
categories: [算法刷题, PAT]
tags: [SPFA, 栈, 队列]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1018 [Public Bike Management](https://pintia.cn/problem-sets/994805342720868352/problems/994805489282433024)

```cpp
#include<iostream>
#include<algorithm>
#include<vector>
#include<queue>
using namespace std;

struct node
{
    int v, w;
    node(int _v, int _w) :v(_v), w(_w) {}
};
const int maxn = 510;
const int inf = 0x3fffffff;
vector<vector<node>> neighbor(maxn);
int bikes[maxn], send[maxn], dis[maxn], pre[maxn], take[maxn];
bool inq[maxn];
int c, n, s, m;
void SPFA(int s)
{
    fill(inq + 1, inq + n + 1, false);
    fill(dis + 1, dis + n + 1, inf);
    fill(send, send + n + 1, 0);
    fill(take, take + n + 1, 0);
    dis[s] = 0;
    queue<int> q;
    q.push(s);
    inq[s] = true;
    while (!q.empty())
    {
        int u = q.front();
        q.pop();
        inq[u] = false;
        for (node n : neighbor[u])
        {
            int v = n.v, w = n.w;
            if (dis[u] + w < dis[v])
            {
                dis[v] = dis[u] + w;
                pre[v] = u;

                if (bikes[v] >= c / 2)
                {
                    send[v] = send[u];
                    take[v] = take[u] + bikes[v] - c / 2;
                }
                else
                {
                    int cur = take[u] - (c / 2 - bikes[v]);
                    if (cur < 0)
                    {
                        send[v] = send[u] - cur;
                        take[v] = 0;
                    }
                    else
                    {
                        send[v] = send[u];
                        take[v] = cur;
                    }
                }

                if (inq[v] == false)
                {
                    q.push(v);
                    inq[v] = true;
                }
            }
            else if (dis[u] + w == dis[v])
            {
                //先算出现这条最短路的send和take
                int curs, curt;
                if (bikes[v] >= c / 2)
                {
                    curs = send[u];
                    curt = take[u] + bikes[v] - c / 2;
                }
                else
                {
                    int cur = take[u] - (c / 2 - bikes[v]);
                    if (cur < 0)
                    {
                        curs = send[u] - cur;
                        curt = 0;
                    }
                    else
                    {
                        curs = send[u];
                        curt = cur;
                    }
                }
                if (curs < send[v] ||(curs == send[v] && curt < take[v]))
                {

                    pre[v] = u;
                    send[v] = curs;
                    take[v] = curt;
                    if (inq[v] == false)
                    {
                        q.push(v);
                        inq[v] = true;
                    }
                }
            }
        }
    }
}
void dfs(int s)
{
    if (s == 0)
    {
        cout << 0;
        return;
    }
    dfs(pre[s]);
    cout << "->" << s;
}

int main()
{
    cin >> c >> n >> s >> m;
    for (int i = 1; i <= n; i++)
    {
        cin >> bikes[i];
    }
    for (int i = 0; i < m; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        neighbor[u].emplace_back(node(v, w));
        neighbor[v].emplace_back(node(u, w));
    }
    SPFA(0);
    cout << send[s] << ' ';
    dfs(s);
    cout << ' ' << take[s];
}
```

    这题之前用dfs做过，代码量也不长，但这次为了熟悉下SPFA算法，就拿这道题试试刀，结果就出问题了，先说最大的问题，测试点7过不去，是因为dis可以用最短路算法，但send和take不能，这个有点复杂，题目要求的是终点send和take大小比较，但最短路的判断过程中，每个点的send和take都是最优的，但局部最优不一定是全局最优，所以如果非要用SPFA，那还得把所有相等的路径存起来，算每条路终点的send和take，进行比较才行，真遇到这种send和take计算比较复杂的，还是用深度优先遍历吧，只有到终点的时候再判断是不是要这条路。SPFA算法注意：

* 我的判断很繁琐，时刻注意更新send和take数组，一旦忘一个，就有可能出错

* 一开始我只想用一个send数组，根据正负来判断送车还是带回车，结果就是拿了20分，其实也不错了哈哈哈，但send这个比较特殊，后面车站的车不能send到前一个

```cpp
#include<iostream>
#include<vector>
using namespace std;
int C, N, Sp, M;//C站点最大车数，N站点数（0,1~N），Sp问题车站编号，M路径数
int bike[501];//每个站点车数
vector<int>v[501];//每个站点的邻居
int dis[501][501];//站点间距离
int minDis[501];
vector<int>path;
vector<int>finalPath;
int finalDis,finalSend,finalTake;
void dfs(int curStation,int curDis,int curSend,int curTake)
{
    //先判断距离是否大于最小距离
    if(curDis > minDis[curStation]) return;
    //将该节点加入路径
    path.emplace_back(curStation);

    //判断是否抵达问题车站
    if(curStation == Sp)
    {
        if ( curDis < minDis[curStation])
        {
            minDis[curStation] = curDis;
            finalDis = curDis;
            finalSend = curSend;
            finalTake = curTake;
            finalPath = path;
        }
        else if (curDis == minDis[curStation])//路径长度相等时
        {
            if(curSend < finalSend)//发送车辆少的优先
            {
                finalSend = curSend;
                finalTake = curTake;
                finalPath = path;
            }
            else if (curSend == finalSend)//发送车辆相同
            {
                if(curTake < finalTake)//带回车辆少的
                {
                    finalTake = curTake;
                    finalPath = path;
                }
            }
        }
    }
    else//如果没有抵达问题车站
    {
        if(curDis < minDis[curStation])minDis[curStation] = curDis;
        for (int i = 0; i < v[curStation].size(); i++)
        {
            int j = v[curStation][i];//邻居编号

            if(bike[j] + curTake <= (C / 2)) dfs(j, curDis + dis[curStation][j],curSend + (C / 2) - bike[j] - curTake,0 );
            else if (bike[j] + curTake > (C / 2))dfs(j, curDis + dis[curStation][j],curSend, bike[j] + curTake - (C / 2));

        }
    }
    path.pop_back();

}
 int main()
 {
     int i,j,k;
     cin>>C>>N>>Sp>>M;
     for ( i = 1;  i <= N; i++)
     {
         cin>>j;
         bike[i] = j;
     }
     while(M--)
     {
         cin>>i>>j>>k;
         dis[i][j] = dis[j][i] = k;//距离矩阵
         v[i].emplace_back(j);
         v[j].emplace_back(i);
     }
     //////////
     for (i = 0; i <= N; i++) minDis[i] = 99999999;
     dfs(0,0,0,0);
     //输出
     cout<<finalSend<<' '<<0;
     for ( i = 1; i < finalPath.size(); i++)
     {
         cout<<"->"<<finalPath[i];
     }
     cout<<' '<<finalTake;

 }
```

    这是dfs版本。

## A1051 [Pop Sequence](https://pintia.cn/problem-sets/994805342720868352/problems/994805427332562944)

```cpp
#include<iostream>
#include<vector>
#include<stack>
using namespace std;

int m, n, k;

int main()
{
    cin >> m >> n >> k;

    for (int i = 0; i < k; i++)
    {
        stack<int> s;
        vector<int> sequence(n);
        for (int j = 0; j < n; j++)
        {
            cin >> sequence[j];
        }
        int order = 1, seIndex = 0;
        bool flag = true;
        for(seIndex = 0; seIndex < n; seIndex++)
        {
            if (s.empty()) s.push(order++);
            while (sequence[seIndex] != s.top() && s.size() < m)
            {
                s.push(order++);
            }

            if (sequence[seIndex] == s.top())
            {
                s.pop();
            }
            else if (s.size() == m)
            {
                cout << "NO" << endl;
                flag = false;
                break;
            }
        }
        if(flag) cout << "YES" << endl;
    }
}
```

    非常朴素的想法，就是用栈来模拟，中间debug了会儿，发现给数组赋值的时候赋错了，`sequence[j]`里面写成i了，怪不得一开始全是NO，数组里只有一个数据，麻了。

## A1056 [Mice and Rice](https://pintia.cn/problem-sets/994805342720868352/problems/994805419468242944)

```cpp
#include<iostream>
#include<vector>
#include<stack>
#include<queue>
#include<algorithm>
using namespace std;

/*
0  1  2  3  4  5 6  7  8  9  10
25 18 0  46 37 3 19 22 57 56 10
6  0  8  7  10 5 9  1  4  2  3
19 25 57 22 10 3 56 18 37 0  46  

5  5  5  2  5  5 5  3  1  3  5
*/
struct player
{
	int w, r;
};
const int maxn = 1010;
vector<player> playerlist(maxn);
int p, g;
int main()
{
	cin >> p >> g;
	for (int i = 0; i < p; i++)
	{
		cin >> playerlist[i].w;
	}
	queue<int> q;
	int groupCnt, matchMem = p;
	for (int i = 0; i < p; i++)
	{
		int order;
		cin >> order;
		q.push(order);
	}
	while (q.size() != 1)
	{
		matchMem = q.size();
		if (matchMem % g == 0) groupCnt = matchMem / g;
		else groupCnt = matchMem / g + 1;
		for (int i = 0; i < groupCnt; i ++)
		{
			int max = q.front();
			playerlist[q.front()].r = groupCnt + 1;
			q.pop();
			for (int j = 1; j < g; j++)
			{
				if (i * g + j >= matchMem) break;
				if (playerlist[q.front()].w > playerlist[max].w)
				{
					max = q.front();
				}
				playerlist[q.front()].r = groupCnt + 1;
				q.pop();
			}
			q.push(max);
		}
	}
	playerlist[q.front()].r = 1;
	for (int i = 0; i < p; i++)
	{
		if (i != 0) cout << ' ';
		cout << playerlist[i].r;
	}
}


```

    队列的一道模拟题，有点绕，有两点非常重要：他给的顺序意思是6号第一个，0号第二个，8号第三个......而不是0号是第6个，1号是第0个，2号是第8个，因为我理解错了，测试样例就没看懂，卡住了。第二处就是，我在想怎么搞排名，要不要用栈来存淘汰的，最后再输出排名，但实际上淘汰的时候就已经知道排名了，只要能够算出组数，那么淘汰的人的名次就是组数+1，这一点很巧妙，省下很多功夫。注意：

* 每一组的名次都可以先赋值成淘汰者的名次，因为晋级的话，会更新的，最后只需要把冠军再单独更新下就行了

* 比的是每个老鼠的重量，而不是编号:(
