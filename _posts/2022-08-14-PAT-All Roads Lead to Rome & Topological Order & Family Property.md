---
title: PAT-All Roads Lead to Rome & Topological Order & Family Property
date: 2022-08-14 22:46:00 +0800
categories: [算法刷题, PAT]
tags: [Dijkstra, 拓扑排序, 并查集]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1087 [All Roads Lead to Rome](https://pintia.cn/problem-sets/994805342720868352/problems/994805379664297984)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<unordered_map>
using namespace std;
//邻接表中需要存的数据：
struct node
{
    int v, cost;//城市编号（map存）、到这个地的花费(其实也就是路径长度)
    node(int _v, int _cost) : v(_v), cost(_cost) {}
};
const int maxn = 210;//最大城市数量
const int inf = 0xfffffff;
bool isVisited[maxn] = { false };//访问数组
vector<vector<int>> pre(maxn);//记录前缀节点
vector<vector<node>> neighbor(maxn);//存邻接表
int cost[maxn];//存到每个城市的最少花费
int happiness[maxn];//存每个城市的幸福指数
int n, k;//城市数目、路数
unordered_map<string, int> map;//名字到编号的映射0~n-1
unordered_map<int, string> map2name;//标号到名字的映射

void Dijkstra(int s)
{
    //初始化数据
    fill(cost, cost + n, inf);//费用初始化为极大值
    cost[s] = 0;//出发点的费用初始化为0
    //遍历n个城市（求到每个城市的最低消费）
    for (int i = 0; i < n; i++)
    {
        int u = -1, min = inf;
        //在没有访问过的节点中，选取cost最低的
        for (int j = 0; j < n; j++)
        {
            if (isVisited[j] == false && cost[j] < min)
            {
                min = cost[j];
                u = j;
            }
        }
        if (u == -1) return;//说明不连通，或者已经遍历完了
        isVisited[u] = true;
        //遍历u的邻居，看是否要更新值
        for (node nd : neighbor[u])
        {
            //u作为中介节点到nd.v节点的花费小于之前存储的最小花费
            if (cost[u] + nd.cost < cost[nd.v])
            {
                cost[nd.v] = cost[u] + nd.cost;//更新花费
                //更新前缀节点
                pre[nd.v].clear();
                pre[nd.v].emplace_back(u);
            }
            else if (cost[u] + nd.cost == cost[nd.v])//如果出现了相同的花费
            {
                pre[nd.v].emplace_back(u);
            }
        }
    }
}
//回溯寻找目标路径
int ans = 0;//路径数量
int tmpHappiness;//临时存储幸福指数和
int pathHappiness;//最终幸福指数和
vector<int> path;//最终的路径
vector<int> tmp;//临时存放路径
//1.确认返回类型和参数
void dfs(int rom)
{
    //2.终止条件
    if (rom == 0)//回到初始点 
    {
        ans++;
        //判断是否要更新为最终选择的路径
        if (ans == 1)//说明这是第一条
        {
            pathHappiness = tmpHappiness;
            path = tmp;
        }
        else if (ans > 1)//说明最小花费的路径大于1条
        {
            //如果总幸福指数更高
            if (tmpHappiness > pathHappiness)
            {
                pathHappiness = tmpHappiness;//更新指数
                path.clear();//更新最终路径
                path = tmp;
            }
            //如果总幸福指数一样，计算平均幸福指数并比较，如果这条路径的平均指数更高
            else if (tmpHappiness == pathHappiness && tmpHappiness / tmp.size() > pathHappiness / path.size())
            {
                pathHappiness = tmpHappiness;//更新指数
                path.clear();//更新最终路径
                path = tmp;
            }
        }
        return;
    }
    //遍历
    for (int p : pre[rom])//遍历每一个前缀节点
    {
        tmp.emplace_back(p);//加入临时路径
        tmpHappiness += happiness[p];//增加临时幸福指数和
        dfs(p);
        tmp.pop_back();//退出路径
        tmpHappiness -= happiness[p];//减少对应的幸福指数
    }
}
int main()
{
    cin >> n >> k;
    string start;//出发点
    cin >> start;
    map[start] = 0;
    map2name[0] = start;
    happiness[0] = 0;
    for (int i = 1; i < n; i++)//此时只有n-1个城市（start算一个了，所以要-1）
    {
        string _city;
        cin >> _city >> happiness[i];
        map[_city] = i;//将该城市写入map，映射城市编号
        map2name[i] = _city;//编号映射城市名字
    }
    //读取路
    for (int i = 0; i < k; i++)
    {
        string _c1, _c2;
        int _cost;
        cin >> _c1 >> _c2 >> _cost;
        neighbor[map[_c1]].emplace_back(node(map[_c2], _cost));
        neighbor[map[_c2]].emplace_back(node(map[_c1], _cost));
    }
    //迪杰斯特拉算法
    Dijkstra(map[start]);
    //寻找目标路径
    tmpHappiness = happiness[map["ROM"]];//提前加入rom的幸福指数
    dfs(map["ROM"]);
    cout << ans << ' ' << cost[map["ROM"]] << ' ' << pathHappiness << ' ' << pathHappiness / path.size() << endl;
    for (int i = path.size() - 1; i >= 0; i--)
    {
        cout << map2name[path[i]] << "->";
    }
    cout << "ROM";
}
```

    这题真是写了好久，使用Dijkstra+dfs，麻了，又又又debug半天，好在我发现在牛客网上，提交出现错误了会告诉样例是啥，不然又不知道要debug多久，需要注意的点：

* 初始化数据的时候，出发点的幸福指数默认应该是0，所以一定要初始化为0，不然一旦加了别的值，应付指数就不对了
* 遍历过的点，一定要记得设置isVisited数组为true
* 题目讲了，如果有多条最短花费路径，那就比总幸福指数，总幸福指数一样了，再比平均幸福指数，注意审题（好在这个点就只有3分，开始没拿到）
* 按我的方法，开始时，需要先加上Rom的幸福指数，我一开始直接加了100，但注意，rom的幸福指数可不一定是100！只是样例中的是100，还好这个不是100的测试点只有6分
* 在打印地名字的时候，注意是`map2name[path[i]]`不是`map2name[i]`
* 在算平均幸福指数时，出发点是不算数的

## A1146 [Topological Order](https://pintia.cn/problem-sets/994805342720868352/problems/994805343043829760)

```cpp
#include<iostream>
#include<vector>
#include<string>
using namespace std;

int n, m, k;//点数、边数、询问数
const int maxv = 1010;
vector<int> inDegree(maxv, 0);//记录每个点的入度
vector<int> tmp;
vector<vector<int>> neighbor(maxv);//邻接表
bool check(vector<int>& order)
{
    tmp = inDegree;
    for (int i = 0; i < n; i++)
    {
        //如果某个节点的入度大于0
        if (tmp[order[i]] > 0) return false;
        //该节点指向的所有点入度-1
        for (int v : neighbor[order[i]])
        {
            tmp[v]--;
        }
    }
    return true;
}
int main()
{
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int v1, v2;
        cin >> v1 >> v2;
        neighbor[v1].emplace_back(v2);
        inDegree[v2]++;
    }
    cin >> k;
    vector<int> ans;//记录不是拓扑排序的序号
    vector<int> order(n);//记录询问的顺序
    for (int i = 0; i < k; i++)
    {
        for (int j = 0; j < n; j++)
        {
            cin >> order[j];
        }
        if (!check(order))
        {
            ans.emplace_back(i);
        }
    }
    for (int i = 0; i < ans.size(); i++)
    {
        if (i == 0) cout << ans[i];
        else cout << ' ' << ans[i];
    }

}
```

    拓扑排序，有一处问题，就是读取数据时，读取order数组，`order[j]`写成`order[i]`了.....这种错误什么时候才能不犯啊......

## A1114 [Family Property](https://pintia.cn/problem-sets/994805342720868352/problems/994805356599820288)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<algorithm>
#include<set>
#include<cstdio>
using namespace std;

struct family
{
    int id, cnt;//家庭最小id，家庭人数，
    double avgSets, avgArea;//平均房产数量、面积
    family(int _id, int _cnt, double _avgSets, double _avgArea) :id(_id), cnt(_cnt), avgSets(_avgSets), avgArea(_avgArea) {}
};
int n;//n行数据
set<int> ids;//储存所有出现过的id
const int maxid = 10000;
int parent[maxid], Rank[maxid], estate[maxid], area[maxid];//父节点、并查集树的高度，某个id（一行数据的代表）房产，某个id的住房面积
//初始化并查集
void init()
{
    for (int i = 0; i < maxid; i++)
    {
        parent[i] = i;
    }
    fill(Rank, Rank + maxid, 0);
}
//查找父节点
int find(int p)
{
    int a = p;
    while (parent[p] != p)
    {
        p = parent[p];
    }
    //路径压缩
    while (parent[a] != a)
    {
        int z = a;
        a = parent[a];
        parent[z] = p;
    }
    return p;
}
//合并两个家庭
void Union(int p, int q)
{
    if (p == -1 || q == -1) return;//不处理-1的情况
    p = find(p); 
    q = find(q);
    //根据rank来看谁做主树，谁高谁做主
    if (Rank[p] > Rank[q])
    {
        parent[q] = p;
    }
    else if (Rank[q] > Rank[p])
    {
        parent[p] = q;
    }
    else//rank一样，p做主
    {
        parent[q] = p;
        Rank[p]++;
    }
}
bool comp(family f1, family f2)
{
    return f1.avgArea != f2.avgArea ? f1.avgArea > f2.avgArea : f1.id < f2.id;
}
int main()
{
    init();//初始化
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        int id, f, m;
        cin >> id >> f >> m;
        ids.insert(id);
        ids.insert(f);
        ids.insert(m);
        Union(id, f);
        Union(id, m);
        //小孩
        int k;
        cin >> k;
        while (k--)
        {
            int kid;
            cin >> kid;
            ids.insert(kid);
            Union(id, kid);
        }
        cin >> estate[id] >> area[id];
    }
    //清除ids中的-1
    ids.erase(-1);
    //遍历每一个ids中的id，找到他们的父亲
    vector<int> tmp[maxid];
    for (int id : ids)
    {
        tmp[find(id)].emplace_back(id);
    }
    vector<family> fa;
    //把tmp数组抽象出来，形成一个家族
    for (int id : ids)
    {
        if (tmp[id].empty()) continue;//说明这个id不是parent
        vector<int>& members = tmp[id];
        int totalEstate = 0, totalArea = 0, familyID = members[0], cnt = members.size();
        for (int m : members)
        {
            if (m < familyID) familyID = m;
            totalEstate += estate[m];//m是某一行数据的代表，并不会每个人都有房产
            totalArea += area[m];
        }
        fa.emplace_back(family(familyID, cnt, (double)totalEstate / (double)cnt, (double)totalArea / (double)cnt));
    }
    sort(fa.begin(), fa.end(), comp);
    printf("%d\n", fa.size());
    for (family& f : fa)
    {
        printf("%04d %d %.3lf %.3lf\n", f.id, f.cnt, f.avgSets, f.avgArea);
    }
}
```

    这道题目是关于并查集的，有点难度，需要注意的点：

* 一定一定要记得init，别光写init函数了，结果再main里没调用

* 要搞清除id之间的关系，在储存某个id的房产时，其实就相当于存了当中某一行的房产

* 注意-1，-1代表没有父母，所以不能union，并且读取完数据，要从set中删除

* 这个解中，用了很多大数组，不要在意，能接算出答案就行了
