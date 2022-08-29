---
title: PAT-Forever & Merging Linked Lists & Postfix Expression & Dijkstra Sequence
date: 2022-08-29 18:12:00 +0800
categories: [算法刷题, PAT]
tags: [素数, 最大公因数, dfs, 排序, 链表, 二叉树的遍历, 后缀表达式, Dijkstra]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    算法笔记基本上刷了一遍了（有些没看，哈夫曼树、欧几里得算法、组合数、提高篇（6）和专题扩展都没看），这几天模拟训练下子，过几天回学校了把没看的看看。

    2h22min91分，四道题都写完了，但第一题和第三题没拿满分，第一题差3分，第三题差6分。最后剩7min的时候，第一题查出问题了，94分，但第三题到最后也没改对。

## A1160 [Forever](https://pintia.cn/problem-sets/994805342720868352/problems/1478635415925800960)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

const int maxn = 10000000;
bool prime[maxn] = { false }, flag;
vector<pair<int, int>> p;
vector<int> digits;
int k, sum;
void Era()
{
    for (int i = 2; i < maxn; i++)
    {
        if (prime[i] == false)
        {
            for (int j = i * 2; j < maxn; j += i)
            {
                prime[j] = true;
            }
        }
    }
}
int gcd(int a, int b)
{
    if (b != 0) return gcd(b, a % b);
    else return a;
}
bool comp(pair<int, int>& p1, pair<int, int>& p2)
{
    if (p1.first != p2.first) return p1.first < p2.first;
    else return p1.second < p2.second;
}
void dfs(int pos, int curSum)
{
    if (curSum > sum || curSum + (k - pos) * 9 < sum || pos > k)
    {
        return;
    }
    else if (pos == k && curSum == sum)
    {
        int n = sum, a = 0;
        for (int i = digits.size() - 1, j = 1; i >= 0; i--, j *= 10)
        {
            a += digits[i] * j;
        }
        int num = a;
        while (a % 10 == 9)
        {
            n -= 9;
            a /= 10;
        }
        n += 1;
        int gcdmn = gcd(sum, n);
        if (gcdmn > 2 && prime[gcdmn] == false)
        {
            flag = true;
            p.emplace_back(make_pair(n, num));
            //cout << n << ' ' << num << endl;;
        }
        return;
    }

    for (int i = 0; i <= 9; i++)
    {
        digits.emplace_back(i);
        dfs(pos + 1, curSum + i);
        digits.pop_back();
    }
}

int main()
{
    Era();
    int N;
    cin >> N;
    for (int i = 1; i <= N; i++)
    {
        flag = false;
        p.clear();
        cin >> k >> sum;
        cout << "Case " << i << endl;
        for (int j = 1; j <= 9; j++)
        {
            digits.emplace_back(j);
            dfs(1, j);
            digits.pop_back();
        }

        if (flag == false)
        {
            cout << "No Solution" << endl;
        }
        else
        {
            sort(p.begin(), p.end(), comp);
            for (int j = 0; j < p.size(); j++)
            {
                cout << p[j].first << ' ' << p[j].second << endl;
            }
        }
    }
}
```

    这道题挺巧的，考察内容很多，所以第一题写了将近50min，考察了素数、最大公因数、dfs和排序。一开始有3分没拿到，就是因为忘排序了，按n升序排，如果出现n一样的再按A来排，还好就一个测试点需要排序。

## A1161 [Merging Linked Lists](https://pintia.cn/problem-sets/994805342720868352/problems/1478635524918910976)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

const int maxn = 100010;
int address2v[maxn], address2next[maxn];
vector<int> l1, l2;

void printList(vector<int>& l1, vector<int>& l2)//l1>l2
{
    int i = 0, j = l2.size() - 1;
    for (; i < l1.size() && j >= 0; i+=2, j--)
    {
        printf("%05d %d %05d\n", l1[i], address2v[l1[i]], l1[i + 1]);
        printf("%05d %d %05d\n", l1[i + 1], address2v[l1[i + 1]], l2[j]);
        if (l1.size() != i+2)
        {
            printf("%05d %d %05d\n", l2[j], address2v[l2[j]], l1[i + 2]);
        }
        else printf("%05d %d -1\n", l2[j], address2v[l2[j]]);
    }
    for (; i < l1.size(); i++)
    {
        if(i != l1.size() -1)
            printf("%05d %d %05d\n", l1[i], address2v[l1[i]], l1[i + 1]);
        else printf("%05d %d -1\n", l1[i], address2v[l1[i]]);
    }


}
int main()
{
    int head1, head2, n;
    cin >> head1 >> head2 >> n;
    while (n--)
    {
        int addr;
        cin >> addr;
        cin >> address2v[addr] >> address2next[addr];
    }
    while (head1 != -1)
    {
        l1.emplace_back(head1);
        head1 = address2next[head1];
    }
    while (head2 != -1)
    {
        l2.emplace_back(head2);
        head2 = address2next[head2];
    }
    if (l1.size() > l2.size())
    {
        printList(l1, l2);
    }
    else
    {
        printList(l2, l1);
    }
}
```

    这个就是链表题，一开始题没审清，按a1->a2->b1->a3->a4->b2的顺序输出了，b数组（短的那个）应该要倒过来输出。

## A1162 [Postfix Expression](https://pintia.cn/problem-sets/994805342720868352/problems/1478635599577522176)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

const int maxn = 25;
bool isRoot[maxn] = { false };
struct node
{
    int left, right;
    string syntax;
}tree[maxn];

void travelsal(int root)
{
    if (root == -1) return;
    cout << '(';
    if (tree[root].left == -1)
    {
        cout << tree[root].syntax;
        travelsal(tree[root].right);
    }
    else
    {
        travelsal(tree[root].left);
        travelsal(tree[root].right);
        cout << tree[root].syntax;
    }
    cout << ')';
}

int main()
{
    int n, root;
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> tree[i].syntax >> tree[i].left >> tree[i].right;
        isRoot[tree[i].left] = isRoot[tree[i].right] = true;
    }
    for (int i = 1; i <= n; i++)
    {
        if (isRoot[i] == false) root = i;
    }
    travelsal(root);
}
```

    这道题亏在不是很懂后缀表达式上了，我看样例里负号是要先打印的，所以我就加了个特判，如果遇到负号，就先序遍历，其他的符号都是后序遍历，但测试点4一直过不去，还6分呢，我一开始猜会不会还有别的符号跟负号一样要先序，但没试出来。

    后来查了啥是后缀表达式，我看人家的负号跟其他负号的安排是一样的，都是后打印，所以应该是这的问题，负号有两种情况，一种是后打印（作为双目运算符），另一种就是先打印，就是取相反数。看了一个博客，他是按左孩子空为条件改先序遍历的，我试了下，果然是这样。如果左孩子是空的，说明这个负号（也有可能是其他啥负号）就要先打印，如果左孩子不空（其实右孩子也就不空了），就要后打印。

## A1163 [Dijkstra Sequence](https://pintia.cn/problem-sets/994805342720868352/problems/1478635670373253120)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

const int maxn = 1010;
const int inf = 0x3fffffff;
int neighbor[maxn][maxn], dis[maxn];
bool vis[maxn] = {false};
int n, m, k;
vector<int> sequence;
bool Dijkstra(int s)
{
    fill(dis, dis + maxn, inf);
    fill(vis, vis + maxn, false);
    dis[s] = 0;
    int index = 0;
    for (int i = 0; i < n; i++)
    {
        int u = -1, min = inf;
        for (int j = 1; j <= n; j++)
        {
            if (vis[j] == false )
            {
                if (dis[j] < min)
                {
                    min = dis[j];
                    u = j;
                }
                else if (dis[j] == min)
                {
                    if (j == sequence[index])
                    {
                        u = j;
                    }
                }
            }
        }
        //if (u == -1) return true;
        if (u != sequence[index]) return false;
        index++;
        vis[u] = true;
        for (int v = 1; v <= n; v++)
        {
            if (vis[v] == false && neighbor[u][v] != inf)
            {
                if (neighbor[u][v] + dis[u] < dis[v])
                {
                    dis[v] = neighbor[u][v] + dis[u];
                }
            }
        }
    }
    return true;
}


int main()
{
    fill(neighbor[0], neighbor[0] + maxn * maxn, inf);
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        neighbor[u][v] = neighbor[v][u] = w;
    }
    cin >> k;
    for (int i = 0; i < k; i++)
    {
        sequence.clear();
        int tmp = 0;
        for (int i = 0; i < n; i++)
        {
            int v;
            cin >> v;
            sequence.emplace_back(v);
        }
        if (Dijkstra(sequence[0])) cout << "Yes" << endl;
        else cout << "No" << endl;
    }
}
```

    这道题我一开始还担心如果每次询问都做一遍Dijkstra会超时，但看来是我多虑了。

    一开始因为担心超时，所以就做一次Dijkstra，记录最短路径长度，去遍历每个序列，先看两两之间是不是有路，在看最后路的长度是不是等于最短路径，但这么做是大错特错的。

    因为所谓Dijkstra序列，指的不是路径而是vis = true的顺序，这个顺序和路径顺序是不一样的，这个vis = true的顺序，相邻两个节点是有可能没路的。所以要稍加修改Dijkstra函数，最后每次都做一遍Dijkstra就行了，也没超时。

    写完这个题还剩38分钟，就去检查之前没过的测试点了。
