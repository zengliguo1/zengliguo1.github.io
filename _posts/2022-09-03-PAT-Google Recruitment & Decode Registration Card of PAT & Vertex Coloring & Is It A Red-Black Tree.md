---
title: PAT-Google Recruitment & Decode Registration Card of PAT & Vertex Coloring & Is It A Red-Black Tree
date: 2022-09-03 17:31:00 +0800
categories: [算法刷题, PAT]
tags: [素数, 模拟, 排序, 哈希表, 红黑树, 二叉树的遍历]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    返校耽搁了两天，昨天在高铁上做了前两道题，又困又累的，状态很差，第一题就拿了15分，第二题没写完，受不了了，就没接着写了。

    因为已经没有连着的4道题了，就选了连着的3道和一道别处的30分的题

    今天在学校把没写完的第二题写完了，后面俩题也写完了，最后剩下五分钟时间，最后一题找不出问题了（实在不太懂红黑树），就去查第一题了，改到了19分，最后总分90分，剩下10分再给时间也找不出来了。

## A1152 [Google Recruitment](https://pintia.cn/problem-sets/994805342720868352/problems/1071785055080476672)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<cmath>
using namespace std;
typedef long long LL;
bool isPrime(LL a)
{
    if (a <= 1) return false;
    LL sqr = (LL)sqrt(a);
    for (LL i = 2; i <= sqr; i++)
    {
        if (a % i == 0) return false;
    }
    return true;
}
int main()
{
    int l, k;
    cin >> l >> k;
    string s;
    cin >> s;
    for (int i = 0; i <= l - k; i++)
    {
        string tmp = s.substr(i, k);
        LL p = stoll(tmp);
        if (isPrime(p))
        {
            cout << tmp;
            return 0;
        }
    }
    cout << 404;
}
```

    这题，一开始我担心超时就用的欧拉筛法，但不能筛太多，太多的话内存不够，所以就拿了15分。最后回来改成普通的查素数方法，竟然有19分，有一个测试点过不去。

    好怪啊，我看了下柳神的代码，跟我的几乎一样，唯一不同的就是在遍历的时候我写的`i <= s.size() - k`，她写的`i <= l - k`，但按理来说`s.size()`和`l`相等啊，奇怪。

## A1153 [Decode Registration Card of PAT](https://pintia.cn/problem-sets/994805342720868352/problems/1071785190929788928)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<cstdio>
#include<unordered_map>
#include<algorithm>
using namespace std;
struct stu
{
    string stuid;
    int score;
    stu(string _id, int _score):stuid(_id), score(_score) {}
};
struct sitenode
{
    int num, people;
    sitenode(int _num, int _people) : num(_num), people(_people) {}
};
const int maxsite = 1000;
const int maxtime = 991300;
int siteN[maxsite] = { 0 }, siteScore[maxsite] = { 0 };
vector<stu> test[3];//ATB
unordered_map<string, unordered_map<int, int> > time2site;
unordered_map<int, int> siteShow;
int n, m, t;
bool comp1(stu s1, stu s2)
{
    if (s1.score != s2.score) return s1.score > s2.score;
    else return s1.stuid < s2.stuid;
}
bool comp2(sitenode s1, sitenode s2)
{
    if (s1.people != s2.people) return s1.people > s2.people;
    else return s1.num < s2.num;
}
int main()
{
    cin >> n >> m;
    for (int i = 0; i < n; i++)
    {
        string sid;
        int ssc;
        cin >> sid >> ssc;
        if (sid[0] == 'A')
        {
            test[0].emplace_back(stu(sid, ssc));
        }
        else if (sid[0] == 'T')
        {
            test[1].emplace_back(stu(sid, ssc));
        }
        else test[2].emplace_back(stu(sid, ssc));
        string site = sid.substr(1, 3);
        int isite = stoi(site);

        string time = sid.substr(4, 6);
        int itime = stoi(time);
        siteN[isite]++;
        siteScore[isite] += ssc;
        time2site[time][isite]++;
    }
    for (int i = 0; i < 3; i++)
    {
        sort(test[i].begin(), test[i].end(), comp1);
    }
    for (int i = 1; i <= m; i++)
    {
        int type;
        cin >> type;
        if (type == 1)
        {
            string s;
            cin >> s;
            printf("Case %d: 1 %s\n", i, s.c_str());
            int index;
            if (s == "A") index = 0;
            else if (s == "T") index = 1;
            else index = 2;
            if (test[index].empty())
            {
                printf("NA\n");
                continue;
            }
            for (stu st : test[index])
            {
                printf("%s %d\n", st.stuid.c_str(), st.score);
            }
        }
        else if (type == 2)
        {
            int s;
            cin >> s;
            printf("Case %d: 2 %d\n", i, s);
            if (siteN[s] == 0)
            {
                printf("NA\n");
                continue;
            }
            printf("%d %d\n", siteN[s], siteScore[s]);
        }
        else
        {
            string s;
            cin >> s;
            t = stoi(s);
            printf("Case %d: 3 %s\n", i, s.c_str());
            if (!time2site.count(s))
            {
                printf("NA\n");
                continue;
            }
            vector<sitenode> type3;
            for (auto it = time2site[s].begin(); it != time2site[s].end(); it++)
            {
                type3.emplace_back(sitenode(it->first, it->second));
            }
            sort(type3.begin(), type3.end(), comp2);
            for (int i = 0; i < type3.size(); i++)
            {
                printf("%d %d\n", type3[i].num, type3[i].people);
            }
        }
    }
}
```

    这道题真是写了好久，得写了一个多小时，可以说最后一题写的时间都没这个长，也可能跟我在高铁上状态不太好有关系吧。需要注意的点挺多的：

* type2问的是某个考场的总人数，这个直接在读取的时候记录起来就行了，但是这个记录下来的数据不能用在tyoe3中，因为同一个考场考试时间可能不同，也就是说type3中某个时间的考场人数可能要比上面考场记录的总人数少

* 想要把考场数据加上时间信息，但不能用数组，因为时间就10^7了，考场再10^3，用数组存的话会内存超限（我试了），所以就用了双层哈希表

* 哈希表的迭代器iterator可以指向键(first)和值(second)

* 看了下柳神的代码，代码量只有我的1/3，她是把所有学生数据存起来，根据type，每次都遍历一遍学生，根据学生的情况分别统计需要的数据。我是只读取的时候就几乎完成了统计工作，只需要后序的判断情况和排序，但还是柳神的简单啊，有时候多遍历几次就遍历几次吧，省了写代码的时间

## A1154 [Vertex Coloring](https://pintia.cn/problem-sets/994805342720868352/problems/1071785301894295552)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<unordered_map>
using namespace std;

const int maxn = 10010;
struct edge
{
    int v, u;
    edge(int _u, int _v) : u(_u), v(_v) {}
};
vector<edge> E;
int n, m;
int main()
{
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int u, v;
        cin >> u >> v;
        E.emplace_back(edge(u, v));
    }
    int q;
    cin >> q;
    while (q--)
    {
        vector<int> vertex(n);
        unordered_map<int, int> color;
        for (int i = 0; i < n; i++)
        {
            cin >> vertex[i];
            color[vertex[i]]++;
        }
        bool flag = true;
        for (edge e : E)
        {
            if (vertex[e.u] == vertex[e.v])
            {
                flag = false;
                break;
            }
        }
        if (flag) cout << color.size() << "-coloring" << endl;
        else cout << "No" << endl;
    }
}
```

    这道题比上道题的代码量小多了，也不难，遍历就完事了。

## A1135 [Is It A Red-Black Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805346063728640)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<cmath>
using namespace std;

struct node
{
    int v;
    node* left = nullptr, * right = nullptr;
    node(int _v) :v(_v) {}
};
vector<int> pre, in;
node* build(int preL, int preR, int inL, int inR)
{
    if (preL > preR) return nullptr;
    node* n = new node(pre[preL]);
    int index;
    for (int i = inL; i <= inR; i++)
    {
        if (in[i] == pre[preL]) index = i;
    }
    n->left = build(preL + 1, index - inL + preL, inL, index - 1);
    n->right = build(index - inL + preL + 1, preR, index + 1, inR);
    return n;
}
bool comp(int a, int b)
{
    return abs(a) < abs(b);
}
int getBlack(node* n)
{
    if (n == nullptr) return 0;
    if (n->v > 0)return 1 + max(getBlack(n->left), getBlack(n->right));
    else return max(getBlack(n->left), getBlack(n->right));
}
bool checkBlack(node* root)
{
    if (root == nullptr) return true;
    bool flag = true;
    if (getBlack(root->left) != getBlack(root->right)) flag = false;
    return flag && checkBlack(root->left) && checkBlack(root->right);
}
bool f;
void travelsal(node* root)
{
    if (root == nullptr) return;
    if (root->v < 0)
    {
        if (root->left && root->left->v < 0) f = false;
        if (root->right && root->right->v < 0) f = false;
    }
    travelsal(root->left);
    travelsal(root->right);
}
int main()
{
    int k, n;
    cin >> k;
    while (k--)
    {
        cin >> n;
        pre.clear();
        in.clear();

        for (int i = 0; i < n; i++)
        {
            int v;
            cin >> v;
            pre.emplace_back(v);
            in.emplace_back(v);
        }
        sort(in.begin(), in.end(), comp);
        node* root;
        root = build(0, n - 1, 0, n - 1);
        f = true;
        travelsal(root);
        if (!checkBlack(root) || root->v < 0 || !f)
        {
            cout << "No" << endl;
            continue;
        }
        cout << "Yes" << endl;
    }
}
```

    这道题，一开始自己写的只拿了21分，后面两个测试点没过去。我一开始是这么判断性质5的：每一层必须颜色一样（就像图一那样），但事实上每一层颜色不一定一样，其实就是遍历每个节点，看它到叶子节点的黑色节点个数，而判断方法和获取树高的方法类似，就看左子树和右子树的树高（此时为黑色节点个数）是否相等即可。

* 注意红黑树不一定是平衡树，其树高可以不平衡
