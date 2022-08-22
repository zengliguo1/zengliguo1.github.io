---
title: PAT-Favorite Color Stripe & Longest Symmetric String & 有向无环图的最长路 & 有向无环图固定终点的最长路
date: 2022-08-22 17:58:00 +0800
categories: [算法刷题, PAT]
tags: [动态规划, 关键路径]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1045 [Favorite Color Stripe](https://pintia.cn/problem-sets/994805342720868352/problems/994805437411475456)

```cpp
#include<iostream>
#include<vector>
using namespace std;

//动规五部曲
//1.确认dp数组以及下标的含义
//以数组A[i]结尾的子序列，其最大长度为dp[i]
//2.确定递推公式：如果i在序列中： dp[i] = max(1, dp[j] + 1) (j = 1... i-1 && color[i] >= color[j])
// 如果i不在序列中dp[i] = 0
//3.dp数组初始化，初始化第一个即可
//4.遍历顺序：从前往后
//5.举例 （pat样例）
//dp数组：1 2 2 3 4 5 6 3 4 5 6 7 

int n, m, l;
const int maxm = 210;
const int maxl = 10010;
vector<int> color(maxm, -1);//color映射成值
vector<int> preColor(maxl, -1);//存储序列的color
vector<int> dp(maxl, 1);
int main()
{
    cin >> n >> m;
    for (int i = 1; i <= m; i++)
    {
        int c;
        cin >> c;
        color[c] = i;
    }
    cin >> l;
    //3.初始化dp数组//已经全初始化为1了

    //4.遍历
    int maxL = 1;
    for (int i = 0; i < l; i++)
    {
        cin >> preColor[i];
        for (int j = 0; j < i; j++)
        {
            if (color[preColor[j]] != -1 && color[preColor[i]] >= color[preColor[j]])
            {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        maxL = max(maxL, dp[i]);
    }
    cout << maxL;
}
```

    这道题目就是按**最长不下降子序列**的方法解，有一处很怪，开始拿了27/30分，去牛客查样例，发现在牛客可以通过，又去网上搜，发现如果只有一个颜色，且这个颜色不在喜欢的里面，最长也要算1，不能算0，麻了。注意：

* 开始出现了段错误，发现是储存序列L的数组开小了，应该是maxl不是maxm

    找到我之前出错的地方了，其实上面这个全初始化为1是不对的，只是恰好能过pat的样例，我之前写的**最长不下降子序列**是这样的：

```cpp
#include<iostream>
#include<vector>
using namespace std;

//动规五部曲
//1.确认dp数组以及下标的含义
//以数组A[i]结尾的子序列，其最大长度为dp[i]
//2.确定递推公式：如果i在序列中： dp[i] = max(1, dp[j] + 1) (j = 1... i-1 && color[i] >= color[j])
// 如果i不在序列中dp[i] = 0
//3.dp数组初始化，初始化第一个即可
//4.遍历顺序：从前往后
//5.举例 （pat样例）
//dp数组：1 2 2 3 4 5 6 3 4 5 6 7 

int n, m, l;
const int maxm = 210;
const int maxl = 10010;
vector<int> color(maxm, -1);//color映射成值
vector<int> preColor(maxl, -1);//存储序列的color
int dp[maxl];
int main()
{
    cin >> n >> m;
    for (int i = 1; i <= m; i++)
    {
        int c;
        cin >> c;
        color[c] = i;
    }
    cin >> l;
    //3.初始化dp数组
    int dp0;
    cin >> dp0;
    dp[0] = color[dp0] == -1 ? 0 : 1;
    preColor[0] = dp0;
    //4.遍历
    int maxL = dp[0];
    for (int i = 1; i < l; i++)
    {
        cin >> preColor[i];
        if (color[preColor[i]] == -1)
        {
            dp[i] = 0;
        }
        else
        {
            dp[i] = 1;
            for (int j = 0; j < i; j++)
            {
                if (color[preColor[j]] != -1 && color[preColor[i]] >= color[preColor[j]])
                {
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
            maxL = max(maxL, dp[i]);
        }
    }
    cout << maxL;
}
```

    在这里，初始化为0是对的，只是我maxL应该初始化为dp[0]，因为dp[0]有可能是1，如果序列长度也是1的话，答案应该是1，但因为我是从1开始遍历，序列长度也是1的话，就不会更新maxL了，自然测试点2就没通过。

```cpp
#include<iostream>
#include<vector>
using namespace std;

//动规五部曲
//1.确认dp数组以及下标的含义
//dp[i][j]表示第一个序列的前i位和第二个序列的前j位，最长子序列的长度
//2.确定递推公式：确定dp[i][j]时，A[i] != B[j] dp[i][j] = max(dp[i-1][j], dp[i][j-1])
// A[i] == B[j] : dp[i][j] = max(dp[i-1][j], dp[i][j-1]) + 1 或者dp[i][j] = dp[i - 1][j - 1] + 1
//3.dp数组初始化，初始化i=0和j=0的所有情况为0
//4.遍历顺序：从前往后
//5.举例 （pat样例）
//dp数组： 1 2 3 4 5 6 7 8 9 10 11 12
//      1 1 2 2 2 2 2 2 2 2 2  2  2 
//      2 1 2 2 2 2 2 2 3 3 3  3  3 
//      3 1 2 2 3 3 3 3 3 4 5  5  5
//      4 1 2 2 3 4 5 5 5 5 5  6  6
//      5 1 2 2 3 4 5 6 6 6 6  6  7

const int maxm = 210;
const int maxl = 10010;
int dp[maxm][maxl];
int favoColor[maxm];
int colorSequence[maxl];
int n, m, l;

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= m; i++)
    {
        cin >> favoColor[i];
    }
    cin >> l;
    for (int i = 1; i <= l; i++)
    {
        cin >> colorSequence[i];
    }
    //初始化dp数组
    for (int i = 0; i < maxm; i++)
    {
        dp[i][0] = 0;
    }
    for (int i = 0; i < maxl; i++)
    {
        dp[0][i] = 0;
    }
    //遍历顺序：从前往后
    for (int i = 1; i <= m; i++)
    {
        for (int j = 1; j <= l; j++)
        {
            if (favoColor[i] != colorSequence[j])
            {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
            else
            {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]) + 1;
                //dp[i][j] = dp[i - 1][j - 1] + 1;
            }
        }
    }
    cout << dp[m][l];
}
```

    这是**最长公共子序列**的方法，这个方法比上一个稍微难一点，涉及到dp二维数组了，但有一点很奇怪，这个dp数组全初始化为0了，但上一个方法必须全初始化为1，按理来讲结果是有区别的，当序列中没有喜欢的颜色时，上一个输出为1，这一个输出为0，但两个方法都能通过pat的测试点，很怪......我知道为啥了......写到这的时候想到了，为啥就写上面了，不在这写了。

## A1040 [Longest Symmetric String](https://pintia.cn/problem-sets/994805342720868352/problems/994805446102073344)

```cpp
#include<iostream>
#include<vector>
#include<string>
using namespace std;

//动规五部曲
//1.确认dp数组以及下标的含义
//dp[i][j]表示i、j之间的字符串是否为回文
//2.确认递推方程
//if s[i] = s[j] dp[i][j] = dp[i+1][j-1]
//else dp[i][j] = 0
//3.dp数组初始化
//初始化所有长度为1和2的子串
//4.遍历顺序
//按子串长度遍历
//5.举例 ABAC
//dp数组： 0 1 2 3
//  i   0 1 0 1 0
//      1   1 0 0
//      2     1 0
//      3       1
//
const int maxs = 1010;
int dp[maxs][maxs];
int main()
{
    string s;
    getline(cin, s);
    int maxL = 1;
    //3.数组初始化
    for (int i = 0; i < s.size(); i++)
    {
        dp[i][i] = 1;
        if (i != s.size() - 1)
        {
            dp[i][i + 1] = s[i] == s[i + 1] ? 1 : 0;
            if (dp[i][i + 1] == 1) maxL = 2;
        }
    }
    if (s.size() <= 2)
    {
        cout << maxL;
        return 0;
    }
    //4.遍历
    for (int l = 3; l <= s.size(); l++)
    {
        for (int i = 0; i <= s.size() - l; i++)
        {
            if (s[i] == s[i + l - 1])
            {
                dp[i][i + l - 1] = dp[i + 1][i + l - 2];
                if (dp[i][i + l - 1] == 1) maxL = l;
            }
            else dp[i][i + l - 1] = 0;
        }
    }
    cout << maxL;
}
```

    最长回文子串，这道题的难点在于遍历的时候，是以子串长度为一个迭代来遍历的，不然会出现动规时，某个dp值的子问题还没有算。

## [有向无环图的最长路](https://sunnywhy.com/sfbj/11/6)

```cpp
#include<iostream>
#include<vector>
#include<string>
using namespace std;

struct edge
{
    int v, w;
    edge(int _v, int _w) : v(_v), w(_w) {}
};
const int maxn = 110;
vector<vector<edge>> neighbor(maxn);
int n, m;
vector<int> dp(maxn, 0);
int DP(int i)
{
    if (dp[i] > 0) return dp[i];
    for (edge e : neighbor[i])
    {
        dp[i] = max(dp[i], DP(e.v) + e.w);
    }
    return dp[i];
}
int main()
{
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        neighbor[u].emplace_back(v, w);
    }
    int maxL = 0;
    for (int i = 0; i < n; i++)
    {
        if (DP(i) > maxL)
        {
            maxL = DP(i);
        }
    }
    cout << maxL;
}
```

    DAG（有向无环图）最长路的动态规划解法，比之前那个方法简洁多了。

## [有向无环图的最长路的最优方案](https://sunnywhy.com/sfbj/11/6/417)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<set>
#include<algorithm>
using namespace std;

struct edge
{
    int v, w;
    edge(int _v, int _w) : v(_v), w(_w) {}
};
const int maxn = 110;
vector<vector<edge>> neighbor(maxn);
vector<int>choice(maxn, -1);
int n, m;
vector<int> dp(maxn, 0);
vector<int> path;
int DP(int i)
{
    if (dp[i] > 0) return dp[i];
    for (edge e : neighbor[i])
    {
        int tmp = max(dp[i], DP(e.v) + e.w);
        if (tmp > dp[i])
        {
            dp[i] = tmp;
            choice[i] = e.v;
        }
    }
    return dp[i];
}
bool comp(edge e1, edge e2)
{
    return e1.v < e2.v;
}
void dfs(int i)
{
    if (choice[i] == -1)//走到尽头
    {
        for (int i = 0; i < path.size(); i++)
        {
            if (i != 0)cout << "->";
            cout << path[i];
        }
        cout << endl;
        return;
    }
    path.emplace_back(choice[i]);
    dfs(choice[i]);
    path.pop_back();
}
int main()
{
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        neighbor[u].emplace_back(v, w);
    }
    for (int i = 0; i < n; i++)
    {
        if (neighbor[i].size() != 0)
        {
            sort(neighbor[i].begin(), neighbor[i].end(), comp);
        }
    }
    int maxL = 0;
    for (int i = 0; i < n; i++)
    {
        if (DP(i) > maxL)
        {
            maxL = DP(i);
        }
    }

    for (int i = 0; i < n; i++)
    {
        if (dp[i] == maxL)
        {
            path.emplace_back(i);
            dfs(i);
            path.pop_back();
        }
    }
}
```

    这里只用打印字典序最小的那个，所以对每个节点的邻接表排了下序，其实用邻接矩阵的话可以规避排序。注意：

* 如果是按字典序打印所有路径的话，使用set来存每个节点后继节点的集合比较方便，同时注意，如果在递归过程中发现了更大的值，要先clear这个set再insert，不然会打印出非最长路径。

## [有向无环图固定终点的最长路](https://sunnywhy.com/sfbj/11/6/418)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<set>
#include<algorithm>
using namespace std;

struct edge
{
    int v, w;
    edge(int _v, int _w) : v(_v), w(_w) {}
};
const int inf = 0xfffffff;
const int maxn = 110;
vector<vector<edge>> neighbor(maxn);
int n, m, target;
vector<int> dp(maxn, -inf);
bool vis[maxn] = { false };
int DP(int i)
{
    if (vis[i]) return dp[i];
    vis[i] = true;
    for (edge e : neighbor[i])
    {
        dp[i] = max(dp[i], DP(e.v) + e.w);
    }
    return dp[i];
}

int main()
{
    cin >> n >> m >> target;
    for (int i = 0; i < m; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        neighbor[u].emplace_back(edge(v, w));
    }
    dp[target] = 0;
    int maxL = -inf;
    for (int i = 0; i < n; i++)
    {
        if (DP(i) > maxL) maxL = DP(i);
    }
    cout << maxL;
}
```

    注意初始化数组变了，其他没变。
