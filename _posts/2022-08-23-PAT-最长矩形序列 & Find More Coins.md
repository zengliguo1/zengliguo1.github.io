---
title: PAT-最长矩形序列 & Find More Coins
date: 2022-08-23 18:46:00 +0800
categories: [算法刷题, PAT]
tags: [动态规划]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## [最长矩形序列](https://sunnywhy.com/sfbj/11/6/419)

```cpp
#include<iostream>
#include<vector>
#include<cmath>
using namespace std;

const int maxn = 110;
vector<int> dp(maxn, 0);//第i个矩形作为最小矩形时，能嵌套的最大矩形序列个数
vector<vector<int>> neighbor(maxn);
int a[maxn];
int b[maxn];
int n;

int DP(int i)
{
    if (dp[i] != 0) return dp[i];
    dp[i] = 1;
    for (int n : neighbor[i])
    {
        dp[i] = max(dp[i], 1 + DP(n));
    }
    return dp[i];
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> a[i] >> b[i];
    }
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (j != i )
            {
                if ((a[i] < a[j] && b[i] < b[j]) || (a[i] < b[j] && b[i] < a[j]))
                {
                    neighbor[i].emplace_back(j);
                }
            }
        }
    }
    int maxL = 0;
    for (int i = 0; i < n; i++)
    {
        maxL = max(maxL, DP(i));
    }
    cout << maxL;
}
```

    和昨天写的关键路径动态规划的思路一样，就是判断邻接关系不一样。

## A1068 [Find More Coins](https://pintia.cn/problem-sets/994805342720868352/problems/994805402305150976)

```cpp
#include<iostream>
#include<vector>
#include<cmath>
#include<algorithm>
using namespace std;

const int maxn = 10010;
const int maxm = 110;
int w[maxn];
int n, m;
vector<vector<int>> dp(maxn, vector<int>(maxm, 0));
vector<vector<int>> choice(maxn, vector<int>(maxm, 0));
//dp[i][v] 金额容量为v时，使用前i个硬币支付的最大金额
vector<int> coin;
void dfs(int i, int j)
{
    if (i <= 0 || j <= 0) return;
    if (choice[i][j] == 1)
    {
        coin.emplace_back(w[i]);
        dfs(i - 1, j - w[i]);
    }
    else
    {
        dfs(i - 1, j);
    }

}
bool comp(int a, int b)
{
    return a > b;
}
int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
    {
        cin >> w[i];
    }
    sort(w + 1, w + n + 1, comp);
    for (int i = 1; i <= n; i++)
    {
        for (int j = w[i]; j <= m; j++)
        {
            if (dp[i - 1][j] > dp[i - 1][j - w[i]] + w[i])
            {
                dp[i][j] = dp[i - 1][j];
                choice[i][j] = 0;
            }
            else
            {
                dp[i][j] = dp[i - 1][j - w[i]] + w[i];
                choice[i][j] = 1;
            }
        }
    }
    if (dp[n][m] != m) cout << "No Solution";
    else
    {
        dfs(n, m);
        for (int i = 0; i < coin.size(); i++)
        {
            if (i != 0)cout << ' ';
            cout << coin[i];
        }
    }
}
```

    动态规划，其实就是01背包问题，但是，这个问题的质量和价值的数组是一致的，开始没看出来，所以也就没想出来。注意，要选小的，那么就从大到小排序，并且在价值相等时，选择当前的硬币，这样就会尽可能地往后选，自然就选了面额小的硬币，很巧妙。
