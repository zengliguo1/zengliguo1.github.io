---
title: PAT-Sexy Primes & Anniversary & Telefraud Detection & Structure of a Binary Tree & find()函数 & sscanf()函数
date: 2022-08-30 20:00:00 +0800
categories: [算法刷题, PAT]
tags: [素数, 哈希表, 模拟, 并查集, 二叉树, 满二叉树]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    今天这次模拟题目有点过山车，前两题有点太简单了，35分钟就ac了前两道题，但是，后两道题简直是噩梦，剩下所有时间都在写这两道题，最后还没写完，又多花了9分钟debug，才把最后一题写完拿了27分，200行真是写麻了，第三题也没全对，所以统计下来，3h拿了64分，3h09min拿了91分，中间上了个厕所，但也没几分钟，估计连两分钟都没，哎。教训就是考试过程一定要滴水不沾，万一就差这两分钟呢。

## A1156 [Sexy Primes](https://pintia.cn/problem-sets/994805342720868352/problems/1478634912848322560)

```cpp
#include<iostream>
#include<cmath>
using namespace std;

bool isPrime(int n)
{
    if (n <= 1) return false;
    int sqr = (int)sqrt(n);
    for (int i = 2; i <= sqr; i++)
    {
        if (n % i == 0) return false;
    }
    return true;
}
int main()
{
    int n;
    cin >> n;
    if (isPrime(n) && isPrime(n - 6))
    {
        cout << "Yes" << endl << n - 6;
    }
    else if (isPrime(n) && isPrime(n + 6))
    {
        cout << "Yes" << endl << n + 6;
    }
    else
    {
        while (1)
        {
            n++;
            if (isPrime(n) && isPrime(n - 6)) break;
            if (isPrime(n) && isPrime(n + 6)) break;
        }
        cout << "No" << endl << n;
    }
}
```

    这道题就是个素数判断题，知道怎么判断素数的话，很快就能写完。

## A1157 [Anniversary](https://pintia.cn/problem-sets/994805342720868352/problems/1478634991737962496)

```cpp
#include<iostream>
#include<string>
#include<unordered_map>
using namespace std;

unordered_map<string, int> alumnus;
string alumnusMin, guestMin = "";

string older(string s1, string s2)
{
    string s11 = s1.substr(6, 8);
    string s22 = s2.substr(6, 8);
    if (stoi(s11) < stoi(s22))
    {
        return s1;
    }
    else return s2;
}

int main()
{
    int n;
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        string s;
        cin >> s;
        alumnus[s]++;
    }
    cin >> n;
    int cnt = 0;
    for (int i = 0; i < n; i++)
    {
        string s;
        cin >> s;
        if (alumnus.count(s))
        {
            cnt++;
            if (cnt == 1) alumnusMin = s;
            else
            {
                alumnusMin = older(alumnusMin, s);
            }
        }
        else
        {
            if (guestMin == "") guestMin = s;
            else
            {
                guestMin = older(guestMin, s);
            }
        }
    }
    if (cnt > 0)
    {
        cout << cnt << endl << alumnusMin;
    }
    else
    {
        cout << cnt << endl << guestMin;
    }

}
```

    这算是一道哈希表的题吧，也没多难，会用string的话很简单。

## A1158 [Telefraud Detection](https://pintia.cn/problem-sets/994805342720868352/problems/1478635060278108160)

```cpp
#include<iostream>
#include<algorithm>
#include<set>
#include<vector>
#include<cstdio>
#include<unordered_map>
#include<cstring>
using namespace std;

const int maxn = 1010;
int connect[maxn][maxn], father[maxn];
int k, n, m;
set<int> suspects;
vector<int> suspects2;
vector<int> realSus;
vector<set<int>> callDiff(maxn);
int FindFather(int x)
{
    int a = x;
    while (x != father[x])
    {
        x = father[x];
    }
    while (a != father[a])
    {
        int z = a;
        a = father[a];
        father[z] = x;
    }
    return x;
}
int main()
{
    cin >> k >> n >> m;
    memset(connect, 0, sizeof(connect));
    for (int i = 0; i < m; i++)
    {
        int p, q, t;
        cin >> p >> q >> t;
        connect[p][q] += t;
        callDiff[p].insert(q);
        if (callDiff[p].size() > k)
        {
            suspects.insert(p);
        }
    }
    for (auto it = suspects.begin(); it != suspects.end(); it++)
    {
        int shortCallCnt = callDiff[*it].size(), cnt = 0;
        for (int s : callDiff[*it])
        {
            if (connect[*it][s] > 5)
            {
                shortCallCnt--;
            }
            else if (connect[s][*it] > 0)
            {
                cnt++;
            }
        }
        if (shortCallCnt > k)
        {
            if (double(cnt) / double(shortCallCnt) <= 0.2)
            {
                realSus.emplace_back(*it);
            }
        }
    }
    for (int i = 0; i < maxn; i++)
    {
        father[i] = i;
    }
    if (realSus.size() == 0)
    {
        cout << "None";
        return 0;
    }
    for (int i = 0; i < realSus.size() - 1; i++)
    {
        for (int j = i + 1; j < realSus.size(); j++)
        {
            int fi = FindFather(realSus[i]);
            int fj = FindFather(realSus[j]);
            if (fi == fj) continue;
            if (connect[realSus[i]][realSus[j]] && connect[realSus[j]][realSus[i]])
            {
                if (fi < fj) father[fj] = fi;
                else father[fi] = fj;
            }
        }
    }
    int gangNum = 0, index = 0;
    unordered_map<int, int> n2index;
    vector<vector<int>> gangs(realSus.size());
    for (int i = 0; i < realSus.size(); i++)
    {
        int p = realSus[i];
        int fp = father[p];
        if (fp == p)
        {
            gangs[index].emplace_back(p);
            n2index[p] = index++;
            gangNum++;
        }
        else
        {
            gangs[n2index[fp]].emplace_back(p);
        }
    }
    if (gangNum == 0) cout << "None";
    else
    {
        for (int i = 0; i < gangNum; i++)
        {
            for (int j = 0; j < gangs[i].size(); j++)
            {
                if (j != 0) cout << ' ';
                cout << gangs[i][j];
            }
            cout << endl;
        }
    }
}
```

    模拟，黑帮检测题，这题描述很多，很复杂，难在数据处理上了，花了很久写完，最后一个测试点没过（6分），本来想着写完第四题再回来debug的，结果第四题直接把时间干完了......

    这道题问题其实挺好发现的，因为我漏了一段话，就是用例1下面的Note没看，他意思是说，一个人给另一个人打多次电话时，要合并成一个来判断是不是短电话，最后一个测试点就是这里。

    看了下柳神的代码，绝了，就五十多行代码。她把每个人的电话时长存在了一个二维数组里，就像邻接矩阵一样，把数据都统计完了，再去判断的，我这是一边判断一边存值，很麻烦。还有就是开辟一个数组后，会默认都赋值为0，bool的话会默认赋值为false。

## A1159 [Structure of a Binary Tree](https://pintia.cn/problem-sets/994805342720868352/problems/1478635126488367104)

    先粘一下我写的屎山代码（其实写完了发现可以用一个flag，但也没时间写了）

```cpp
#include<iostream>
#include<vector>
#include<string>
using namespace std;

struct node
{
    int v;
    node* left = nullptr, * right = nullptr;
    node(int _v) : v(_v) {}
}*root;
const int maxn = 32;
vector<int> post(maxn);
vector<int> in(maxn);
node* build(int pL, int pR, int iL, int iR)
{
    if (pL > pR) return nullptr;
    node* n = new node(post[pR]);
    int index;
    for (int i = iL; i <= iR; i++)
    {
        if (in[i] == post[pR])
        {
            index = i;
            break;
        }
    }
    n->left = build(pL, pL + index - 1 - iL, iL, index - 1);
    n->right = build(pL + index - iL, pR - 1, index + 1, iR);
    return n;
}
void case1(int v)
{
    if (root->v == v)
    {
        cout << "Yes" << endl;
    }
    else cout << "No" << endl;
}
bool flag2, flag3, flag4, flag5, flag7;
void travelsal2(node* root, int v1, int v2)
{
    if (root == nullptr) return;
    if (root->left && root->right)
    {
        if ((root->left->v == v1 && root->right->v == v2) || (root->left->v == v2 && root->right->v == v1))
        {
            flag2 = true;
            return;
        }
    }
    travelsal2(root->left, v1, v2);
    travelsal2(root->right, v1, v2);
}
void case2(int v1, int v2)
{
    flag2 = false;
    travelsal2(root, v1, v2);
    if (flag2) cout << "Yes" << endl;
    else cout << "No" << endl;
}
void travelsal3(node* root, int p, int c)
{
    if (root == nullptr) return;
    if (root->v == p)
    {
        if (root->left && root->left->v == c)
        {
            flag3 = true;
            flag4 = true;
        }
        else if (root->right && root->right->v == c)
        {
            flag3 = true;
            flag5 = true;
        }

    }
    travelsal3(root->left, p, c);
    travelsal3(root->right, p, c);
}
void case3(int p, int c)
{
    flag3 = false;
    travelsal3(root, p, c);
    if (flag3) cout << "Yes" << endl;
    else cout << "No" << endl;
}
void case4(int p, int c)
{
    flag4 = false;
    travelsal3(root, p, c);
    if (flag4) cout << "Yes" << endl;
    else cout << "No" << endl;
}
void case5(int p, int c)
{
    flag5 = false;
    travelsal3(root, p, c);
    if (flag5) cout << "Yes" << endl;
    else cout << "No" << endl;
}
int l1, l2;
void getLevel(node* root, int level, int n1, int n2)
{
    if (root == nullptr) return;
    if (root->v == n1) l1 = level;
    else if (root->v == n2) l2 = level;
    getLevel(root->left, level + 1, n1, n2);
    getLevel(root->right, level + 1, n1, n2);
}
void case6(int n1, int n2)
{
    getLevel(root, 0, n1, n2);
    if (l1 == l2)cout << "Yes" << endl;
    else cout << "No" << endl;
}
void travelsal7(node* root)
{
    if (root == nullptr) return;
    if (root->left && !root->right) flag7 = false;
    else if (root->right && !root->left) flag7 = false;
    travelsal7(root->left);
    travelsal7(root->right);
}
void case7()
{
    flag7 = true;
    travelsal7(root);
    if (flag7) cout << "Yes" << endl;
    else cout << "No" << endl;
}
int main()
{
    int n;
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> post[i];
    }
    for (int i = 0; i < n; i++)
    {
        cin >> in[i];
    }
    root = build(0, n - 1, 0, n - 1);
    int m;
    cin >> m;
    while (m--)
    {
        int CASE;
        string s[4], stmp;
        for (int i = 0; i < 4; i++)
        {
            cin >> s[i];
        }

        if (s[3] == "root") CASE = 1;
        else if (s[3] == "are")
        {
            cin >> stmp;
            if (stmp == "siblings") CASE = 2;
            else
            {
                CASE = 6;
                getline(cin, stmp);
            }
        }
        else if (s[3] == "parent")
        {
            CASE = 3;
            cin >> s[1]; cin >> s[1];

        }
        else if (s[3] == "left")
        {
            CASE = 4;
            cin >> s[1]; cin >> s[1]; cin >> s[1];
        }
        else if (s[3] == "right")
        {
            CASE = 5;
            cin >> s[1]; cin >> s[1]; cin >> s[1];
        }
        else
        {
            CASE = 7;
            cin >> s[1];
        }

        switch (CASE)
        {
        case 1:
        {
            int root = stoi(s[0]);
            case1(root);
            break;
        }
        case 2:
        {
            int v1 = stoi(s[0]), v2 = stoi(s[2]);
            case2(v1, v2);
            break;
        }
        case 3:
        {
            int p = stoi(s[0]), c = stoi(s[1]);
            case3(p, c);
            break;
        }
        case 4:
        {
            int p = stoi(s[1]), c = stoi(s[0]);
            case4(p, c);
            break;
        }
        case 5:
        {
            int p = stoi(s[1]), c = stoi(s[0]);
            case5(p, c);
            break;
        }
        case 6:
        {
            int n1 = stoi(s[0]), n2 = stoi(s[2]);
            case6(n1, n2); break;
        }
        case 7:
        {
            case7();
            break;
        }
        default:
            break;
        }

    }
}
```

    这两百多行的代码，也没拿满分，差三分，我也不打算找bug了，直接重构好吧，这两百行在考试的情况下是必不可能写的，太浪费时间了。下面是优化的代码：

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<unordered_map>
using namespace std;

struct node
{
    int v, level;
    node* left = nullptr, * right = nullptr, *parent = nullptr;
    node(int _v) : v(_v) {}
}*root;
const int maxn = 32;
vector<int> post(maxn);
vector<int> in(maxn);
bool isFull = true;
unordered_map<int, node*> v2node;
node* build(int pL, int pR, int iL, int iR)
{
    if (pL > pR) return nullptr;
    node* n = new node(post[pR]);
    int index;
    for (int i = iL; i <= iR; i++)
    {
        if (in[i] == post[pR])
        {
            index = i;
            break;
        }
    }
    n->left = build(pL, pL + index - 1 - iL, iL, index - 1);
    n->right = build(pL + index - iL, pR - 1, index + 1, iR);
    if (n->left) n->left->parent = n;
    if (n->right) n->right->parent = n;
    v2node[n->v] = n;
    return n;
}
void dfs(node* root)
{
    if (root == nullptr) return;
    if (root->parent)root->level = root->parent->level + 1;
    if (root->left && !root->right)isFull = false;
    if (root->right && !root->left)isFull = false;
    dfs(root->left);
    dfs(root->right);
}
int main()
{
    int n;
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> post[i];
    }
    for (int i = 0; i < n; i++)
    {
        cin >> in[i];
    }
    root = build(0, n - 1, 0, n - 1);
    root->level = 1;
    dfs(root);
    int m;
    cin >> m;
    getchar();
    while (m--)
    {
        bool flag = false;
        int a, b;
        string s;
        getline(cin, s);
        if (s.find("root") != string::npos)
        {
            sscanf(s.c_str(), "%d is the root", &a);
            flag = root->v == a ? true : false;
        }
        else if (s.find("siblings") != string::npos)
        {
            sscanf(s.c_str(), "%d and %d are siblings", &a, &b);
            flag = v2node[a]->parent == v2node[b]->parent ? true : false;
        }
        else if (s.find("parent") != string::npos)
        {
            sscanf(s.c_str(), "%d is the parent of %d", &a, &b);
            flag = v2node[a] == v2node[b]->parent ? true : false;
        }
        else if (s.find("left") != string::npos)
        {
            sscanf(s.c_str(), "%d is the left child of %d", &a, &b);
            flag = v2node[b]->left == v2node[a] ? true : false;
        }
        else if (s.find("right") != string::npos)
        {
            sscanf(s.c_str(), "%d is the right child of %d", &a, &b);
            flag = v2node[b]->right == v2node[a] ? true : false;
        }
        else if (s.find("level") != string::npos)
        {
            sscanf(s.c_str(), "%d and %d are on the same level", &a, &b);
            flag = v2node[a]->level == v2node[b]->level ? true : false;
        }
        else
        {
            flag = isFull;
        }
        if (flag) cout << "Yes" << endl;
        else cout << "No" << endl;
    }
}
```

    这个优化版的得益于string的find函数，之前知道这个函数，但没用过，所以没敢用，结果就是写出了屎山。find还是很好使的，find不到了就会返回`string::npos`。

    还有一个函数，之前从来没用过，就是`sscanf`，可以像scanf那样读取一个字符串，这个字符串可以设置为已有的字符串，就是得是char数组，只要用`c_str()`转化下就行了。
