---
title: PAT-Heaps & Is It a Complete AVL Tree & Complete Binary Tree & Lowest Common Ancestor
date: 2022-08-16 17:39:00 +0800
categories: [算法刷题, PAT]
tags: [堆, AVL, 完全二叉树, LCA]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1147 [Heaps](https://pintia.cn/problem-sets/994805342720868352/problems/994805342821531648)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;
const int maxn = 1010;
int n, m;
vector<int> tree(maxn, -1);
vector<int> post;

bool checkMinHeap(int root)
{
    if (root > m) return true;
    bool flag = true;
    if (root * 2 <= m && tree[root * 2] < tree[root])
    {
        flag = false;
    }
    if (root * 2 + 1 <= m && tree[root * 2 + 1] < tree[root])
    {
        flag = false;
    }
    return flag && checkMinHeap(root * 2) && checkMinHeap(root * 2 + 1);
}
bool checkMaxHeap(int root)
{
    if (root > m) return true;
    bool flag = true;
    if (root * 2 <= m && tree[root * 2] > tree[root])
    {
        flag = false;
    }
    if (root * 2 + 1 <= m && tree[root * 2 + 1] > tree[root])
    {
        flag = false;
    }
    return flag && checkMaxHeap(root * 2) && checkMaxHeap(root * 2 + 1);
}


void postOrder(int root)
{
    if (root > m) return;
    postOrder(root * 2);
    postOrder(root * 2 + 1);
    post.emplace_back(tree[root]);
}
void printPost()
{
    for (int i = 0; i < post.size(); i++)
    {
        if (i == 0) cout << post[i];
        else cout << ' ' << post[i];
    }
    cout << endl;
}
int main()
{
    //freopen("input.txt", "r", stdin);
    cin >> n >> m;
    for (int i = 0; i < n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            cin >> tree[j];
        }
        post.clear();
        if (tree[1] - tree[2] >= 0)
        {
            if (checkMaxHeap(1))
            {
                cout << "Max Heap" << endl;
            }
            else
            {
                cout << "Not Heap" << endl;
            }
        }
        else
        {
            if (checkMinHeap(1))
            {
                cout << "Min Heap" << endl;
            }
            else
            {
                cout << "Not Heap" << endl;
            }
        }
        postOrder(1);
        printPost();
    }

}
```

    堆的判定，比建堆要简单，23分52秒93拿下，注意点：

* 用数组存堆，从1开始，这样左孩子就是root\*2，右孩子就是root\*2+1

* 中间打印出问题，是因为遍历时，把root加到数组中了，应该加tree[root]

## A1123 [Is It a Complete AVL Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805351302414336)

```cpp
#include<iostream>
#include<vector>
#include<queue>
using namespace std;
int n;
vector<int> levelSequence;
struct node
{
    int val;
    int height = 1;
    node* left;
    node* right;
    node(int  _val) : val(_val), left(nullptr), right(nullptr) {}
};
int GetHeight(node*& root)
{
    if (root == nullptr) return 0;
    else return root->height;
}
void UpdateHeight(node*& root)
{
    root->height = max(GetHeight(root->left), GetHeight(root->right)) + 1;
}
int GetBalanceFactor(node*& root)
{
    return GetHeight(root->left) - GetHeight(root->right);
}

void R(node*& root)
{
    node* tmp = root ->left;
    root->left = tmp->right;
    tmp->right = root;
    root = tmp;
    UpdateHeight(root->right);
    UpdateHeight(root);
}
void L(node*& root)
{
    node* tmp = root->right;
    root->right = tmp->left;
    tmp->left = root;
    root = tmp;
    UpdateHeight(root->left);
    UpdateHeight(root);
}
void Insert(node*& root, int val)
{
    if (root == nullptr)
    {
        root = new node(val);
        return;
    }

    if (val < root->val)
    {
        Insert(root->left, val);
        UpdateHeight(root);
        if (GetBalanceFactor(root) == 2)//L
        {
            if (GetBalanceFactor(root->left) == 1)//LL
            {
                R(root);
            }
            else//LR
            {
                L(root->left);
                R(root);
            }
        }
    }
    else if(val > root->val)
    {
        Insert(root->right, val);
        UpdateHeight(root);
        if (GetBalanceFactor(root) == -2)//R
        {
            if (GetBalanceFactor(root->right) == -1)//RR
            {
                L(root);
            }
            else//RL
            {
                R(root->right);
                L(root);
            }
        }
    }
}
void PrintLevel()
{
    for (int i = 0; i < levelSequence.size(); i++)
    {
        if (i == 0)cout << levelSequence[i];
        else cout << ' ' << levelSequence[i];
    }
    cout << endl;
}
void LevelTravelsal(node*& root)
{
    bool isComp = true;
    bool flag = false;
    queue<node*> q;
    q.push(root);
    while (!q.empty())
    {
        node* top = q.front();
        q.pop();
        levelSequence.emplace_back(top->val);
        if (top->left == nullptr)
        {
            flag = true;
        }
        else
        {
            if (flag) isComp = false;
            q.push(top->left);
        }

        if (top->right == nullptr)
        {
            flag = true;
        }
        else
        {
            if (flag)isComp = false;
            q.push(top->right);
        }
    }
    PrintLevel();
    if (isComp)
    {
        cout << "YES";
    }
    else
    {
        cout << "NO";
    }
}

int main()
{
    //freopen("input.txt", "r", stdin);
    cin >> n;
    node* root = nullptr;
    for (int i = 0; i < n; i++)
    {
        int val;
        cin >> val;
        Insert(root, val);
    }
    LevelTravelsal(root);
}
```

    1个小时，拿到10分......这道题出现了三个问题，一个比较大的问题是不会判断完全二叉树......我一开始自己写的只能判断一些情况。

- 完全二叉树直接在层序遍历的时候去判断，如果在入队前，发现有了空的左孩子或者右孩子，此时就要将flag打开，如果之后再遍历到某个节点，发现它有左孩子或者右孩子时，就说明它不再是一个完全二叉树

- 第二个错就是，左旋的时候更新树高时，更新错了，应该是`UpdateHeight(root)`，我写成了`UpdateHeight(root->right)`，此时root->right是空的，自然也就没有height，就中断了，也就是段错误

- 第三个错，就是新建节点时，树高要设置为1，不然有些不平衡的情况，漏掉旋转

## 1110 [Complete Binary Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805359372255232)

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<queue>
using namespace std;

const int maxn = 20;
vector<int> Left(maxn,-1);
vector<int> Right(maxn, -1);
vector<bool> isRoot(maxn, true);
int n;
int main()
{
    //freopen("input.txt", "r", stdin);
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        string l, r;
        cin >> l >> r;
        if (l[0] != '-')
        {
            Left[i] = stoi(l);
            isRoot[stoi(l)] = false;
        }
        if (r[0] != '-')
        {
            Right[i] = stoi(r);
            isRoot[stoi(r)] = false;
        }
    }
    int root;
    for (int i = 0; i < n; i++)
    {
        if (isRoot[i] == true) root = i;
    }
    queue<int> q;
    q.push(root);
    bool flag = false, isComp = true;
    int last;
    while (!q.empty())
    {
        int top = q.front();
        q.pop();
        if (Left[top] == -1)
        {
            flag = true;
        }
        else
        {
            if (flag) isComp = false;
            q.push(Left[top]);
        }
        if (Right[top] == -1)
        {
            flag = true;
        }
        else
        {
            if (flag) isComp = false;
            q.push(Right[top]);
        }
        if (q.empty()) last = top;
    }
    if (isComp)
    {
        cout << "YES " << last;
    }
    else
    {
        cout << "NO " << root;
    }
}
```

    20分钟16分，30分钟debug+查答案拿到25分。这道题的判断完全二叉树的方法用了上一题的方法，过程中出现了答案错误和内存超限。

* 在读取l和r的时候，我一开始设置的数据类型是char，但n的个数最多有20个，也就是说，可能有的节点是两位数的编号，而char就只能存一位，自然就出错了。一开始我还想着用string和stoi来转换，但寻思这不纯纯浪费么，char就行了，然鹅事与愿违，忽略了这个点，麻了

## A1143 [Lowest Common Ancestor](https://pintia.cn/problem-sets/994805342720868352/problems/994805343727501312)

```cpp
#include<iostream>
#include<vector>
#include<unordered_map>
#include<cstdio>
using namespace std;
unordered_map<int, int> map;
const int maxn = 10010;
vector<int> preOrder(maxn);
int m, n;
void FindLCA(int u, int v)
{
    for (int i = 0; i < n; i++)
    {
        if ((preOrder[i] > u && preOrder[i] < v) || (preOrder[i] < u && preOrder[i] > v))
        {
            printf("LCA of %d and %d is %d.\n", u, v, preOrder[i]);
            return;
        }
        else if (preOrder[i] == u)
        {
            printf("%d is an ancestor of %d.\n", u, v);
            return;
        }
        else if (preOrder[i] == v)
        {
            printf("%d is an ancestor of %d.\n", v, u);
            return;
        }
    }
}
int main()
{
    cin >> m >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> preOrder[i];
        map[preOrder[i]]++;
    }
    for (int i = 0; i < m; i++)
    {
        int x, y;
        cin >> x >> y;
        if (!map.count(x) && !map.count(y))
        {
            printf("ERROR: %d and %d are not found.\n", x, y);
        }
        else if (!map.count(x))
        {
            printf("ERROR: %d is not found.\n", x);
        }
        else if (!map.count(y))
        {
            printf("ERROR: %d is not found.\n", y);
        }
        else
        {
            FindLCA(x, y);
        }
    }
}
```

    这道题根本就没建树，巧妙地利用了平衡二叉树和前序遍历地性质，如果是公共祖先的话，根据avl的性质，祖先的左侧的节点值都要小于祖先右侧的节点值，所以如果有公共祖先，那么其值一定介于两个节点之间。其次，由于是前序遍历，所以祖先一定会先遍历到，所以只要遍历前序数组即可找到祖先。如果某一方是另一方的祖先，那么只要判断在前序遍历中先遇到谁，谁就是祖先，毕竟前序遍历顺序是根->左->右。
