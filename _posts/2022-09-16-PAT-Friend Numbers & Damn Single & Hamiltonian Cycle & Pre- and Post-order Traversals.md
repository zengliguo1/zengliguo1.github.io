---
title: PAT-Friend Numbers & Damn Single & Hamiltonian Cycle & Pre- and Post-order Traversals
date: 2022-09-16 16:07:00 +0800
categories: [算法刷题, PAT]
tags: [模拟, 哈希表, 哈密顿图, 二叉树的遍历, 树的构造]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

    昨天发现浙软通知19号下午1点半到3点半机试，还是用OMS考，剩下三天，每天这个点刷一套题吧。

    今天1点半准时开始，前三道题50分钟就写完了，心想还挺简单，做最后一题就不这么想了，最后一题开始写了1小时10分钟，对了26分，差4分，后来又改了8分钟，拿满了，也就是说，今日战绩：2h18min 100分。

## A1120 [Friend Numbers](https://pintia.cn/problem-sets/994805342720868352/problems/994805352925609984)

```cpp
#include<iostream>
using namespace std;
int id[40], n, cnt;

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        int num, sum = 0;
        cin >> num;
        while (num != 0)
        {
            sum += (num % 10);
            num /= 10;
        }
        id[sum]++;
    }
    for (int i : id)
    {
        if (i != 0) cnt++;
    }
    cout << cnt << endl;
    bool flag = false;
    for (int i = 0; i < 40; i++)
    {
        if (id[i] > 0)
        {
            if (flag)
            {
                cout << ' ';
            }
            cout << i;
            flag = true;
        }
    }
}
```

    简单的模拟。

## A1121 [Damn Single](https://pintia.cn/problem-sets/994805342720868352/problems/994805352359378944)

```cpp
#include<iostream>
#include<unordered_map>
#include<cstdio>
#include<vector>
#include<set>
using namespace std;
const int maxn = 100000;
int come[maxn], n, m, p, q;
unordered_map<int, int> cp;
set<int> single;
vector<int> allPeople;
int main()
{
    cin >> n;
    while (n--)
    {
        cin >> p >> q;
        cp[p] = q;
        cp[q] = p;
    }
    cin >> m;
    while (m--)
    {
        cin >> p;
        allPeople.emplace_back(p);
        come[p] = 1;
    }
    for (int i = 0; i < allPeople.size(); i++)
    {
        if (!cp.count(allPeople[i]) || come[cp[allPeople[i]]] == 0)
        {
            single.insert(allPeople[i]);
        }
    }
    printf("%d\n", single.size());
    for (auto i = single.begin(); i != single.end(); i++)
    {
        if (i != single.begin()) printf(" ");
        printf("%05d", *i);
    }
}
```

    哈希表，其实想考排序，但我直接用set了，规避掉了排序。注意要打印五位id，不然有一个测试点过不去（竟然就只有一个）。

## A1122 [Hamiltonian Cycle](https://pintia.cn/problem-sets/994805342720868352/problems/994805351814119424)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<unordered_map>
using namespace std;
const int maxn = 201;
int N, m, k, n, edge[maxn][maxn], u, v, i;
int main()
{
    cin >> N >> m;
    while (m--)
    {
        cin >> u >> v;
        edge[u][v] = edge[v][u] = 1;
    }
    cin >> k;
    while (k--)
    {
        cin >> n;
        if (n != (N + 1))
        {
            cout << "NO" << endl;
            string s;
            getline(cin, s);
            continue;
        }
        vector<int> vertex(n);
        unordered_map<int, int> map;
        for ( i = 0; i < n; i++)
        {
            cin >> vertex[i];
            map[vertex[i]]++;
        }
        if (vertex.front() != vertex.back() || map.size() < N)
        {
            cout << "NO" << endl;
            continue;
        }
        for ( i = 0; i < n - 1; i++)
        {
            if (edge[vertex[i]][vertex[i + 1]] == 0) break;
        }
        if (i == n - 1) cout << "YES" << endl;
        else cout << "NO" << endl;
    }
}
```

    判断是否为哈密顿图，哈密顿图是每个节点都经过一次最后返回原点的圈。又犯了老毛病，开始判断节点不是`N+1`后直接`continue`了，这样是不对的，后面节点的信息没有读取，会影响下一条路的判断。所以要用getline给吸收掉。

## A1119 [Pre- and Post-order Traversals](https://pintia.cn/problem-sets/994805342720868352/problems/994805353470869504)

```cpp
#include<iostream>
#include<vector>
using namespace std;
int n, pre[30], post[30];
vector<int> in;
struct node
{
    int v;
    node* left = nullptr, * right = nullptr;
    node(int _v) : v(_v) {}
};
bool flag = true;
node* build(int preL, int preR, int postL, int postR)
{
    if (preL > preR) return nullptr;
    node* n = new node(pre[preL]);
    if (preL == preR) return n;
    if (pre[preL + 1] == post[postR - 1])
    {
        flag = false;
        n->left = build(preL + 1, preR, postL, postR - 1);
        return n;
    }
    int postLR, preRL = -1;
    for (int i = postL; i <= postR ; i++)
    {
        if (pre[preL + 1] == post[i])
        {
            postLR = i;
            break;
        }
    }
    for (int i = preL; i <= preR; i++)
    {
        if (pre[i] == post[postR - 1])
        {
            preRL = i;
            break;
        }
    }
    n->left = build(preL + 1, preRL - 1, postL, postLR);
    n->right = build(preRL, preR, postLR + 1, postR - 1);
    return n;
}
void inorder(node* root)
{
    if (root == nullptr) return;
    inorder(root->left);
    in.emplace_back(root->v);
    inorder(root->right);
}
int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> pre[i];
    }
    for (int i = 0; i < n; i++)
    {
        cin >> post[i];
    }
    node* root;
    root = build(0, n - 1, 0, n - 1);
    if (flag) cout << "Yes" << endl;
    else cout << "No" << endl;
    inorder(root);
    for (int i = 0; i < in.size(); i++)
    {
        if (i != 0) cout << ' ';
        cout << in[i];
    }
    cout << endl;
}
```

    这道题挺复杂，给前序和后序，要建一棵树，但这棵树有可能不是唯一的，什么时候不唯一呢，就是当要建当前节点的左右子树时，发现前序数组的`pre[preL + 1]`等于后序数组的`post[postR - 1]`时，此时，既可以建造根节点值为`pre[preL + 1]`的`pre[preL]`的左子树或者建造右子树（这里有点绕），我统一建成了左子树。

    一开始没拿满分是因为我以为只有像样例2那样，最后剩两个节点了，才会出现不唯一的情况。PAT数据确实挺照顾人的，就一个测试点不是剩两个节点。
