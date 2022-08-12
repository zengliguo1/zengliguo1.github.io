---
title: PAT-Build A Binary Search Tree & Root of AVL Tree & Social Clusters
date: 2022-08-10 23:38:00 +0800
categories: [算法刷题, PAT]
tags: [BST, AVL, 并查集]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1099 [Build A Binary Search Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805367987355648)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<queue>
using namespace std;

struct node
{
    int val, left, right;
};
const int maxn = 100;
vector<node> bst(maxn);
vector<int> keys(maxn);
int n;//树的节点总数
int index = 0;//从0开始赋值
vector<int> levelkey;
void inTravelsal(int root)
{
    //终止条件
    if (root >= n || root == -1) return;//说明这个节点不存在
    //中序遍历
    inTravelsal(bst[root].left);
    bst[root].val = keys[index++];
    inTravelsal(bst[root].right);
}
void levelTravelsal(int root)
{
    queue<int> q;
    q.push(root);
    while (!q.empty())
    {
        int top = q.front();
        q.pop();
        levelkey.emplace_back(bst[top].val);
        //把该节点的孩子都放入队列中
        if (bst[top].left != -1) q.push(bst[top].left);
        if (bst[top].right != -1) q.push(bst[top].right);
    }
}
int main()
{
    cin >> n;
    //读取树的结构
    for (int i = 0; i < n; i++)
    {
        cin >> bst[i].left;
        cin >> bst[i].right;
    }
    //读取键值
    for (int i = 0; i < n; i++)
    {
        cin >> keys[i];
    }
    sort(keys.begin(), keys.begin() + n);
    //中序遍历为树的每个节点来赋值
    inTravelsal(0);
    //层序遍历输出答案
    levelTravelsal(0);
    //输出答案
    for (int i = 0; i < n; i++)
    {
        if (i == 0) cout << levelkey[i];
        else cout << ' ' << levelkey[i];
    }
}
```

    这道题目就是中序遍历建二叉搜索树，再层序遍历输出，需要注意的地方有两点：

* sort时，由于我初始化数组，给了数组maxn的空间，所以排序时**不能**简单地调用`sort(keys.begin(), keys.end())`，要明确排序地范围

* 在中序遍历时，终止条件要记得加上`root != -1`否则会提示数组越界

## A1066[Root of AVL Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805404939173888)

```cpp
#include<iostream>
#include<vector>
using namespace std;
//AVL树的节点
struct node
{
    int height = 1;//默认新建一个节点的高度是1
    int val;//节点的权值
    node* left = nullptr, * right = nullptr;//左右孩子，默认是空值
    node(int _val) :val(_val) {}
};
//获取某个节点的高度（包括空节点）
int GetHeight(node*& root)
{
    if (root == nullptr) return 0;
    else return root->height;
}
//实现获取当前平衡因子的函数
int BalanceFactor(node*& root)
{
    return GetHeight(root->left) - GetHeight(root->right);
}
void UpdateHeight(node*& root)
{
    root->height = max(GetHeight(root->left), GetHeight(root->right)) + 1;
}
//实现AVL树右旋
void R(node*& root)
{
    node* tmp = root->left;//存下root的左子树指针
    //root的left更新为其左子树的右子树
    root->left = tmp->right;
    //root左子树的right指向root
    tmp->right = root;
    //旋转后要更新两个节点的高度
    UpdateHeight(root);//先更新低的
    UpdateHeight(tmp);//再更新高的
    //将root更新为tmp
    root = tmp;
}
//实现AVL树左旋
void L(node*& root)
{
    node* tmp = root->right;//储存root的right指针
    //root的right要更新为其右子树的左子树
    root->right = tmp->left;
    //root右子树的left要指向root
    tmp->left = root;
    //更新两个节点的高度
    UpdateHeight(root);//先更新低的
    UpdateHeight(tmp);
    //将root更新
    root = tmp;

}

//实现插入节点的函数
void Insert(node*& root, int val)
{
    if (root == nullptr)//当前递归到的节点为空时，在此处插入节点
    {
        node* n = new node(val);
        root = n;
        return;
    }
    //如果不空，说明还要继续寻找合适的位置
    if (val < root->val)//如果要插入的值小于当前节点的值
    {
        //那么就要插入该节点的左孩子处
        Insert(root->left, val);
        //更新root的高度(因为是插入了root的左孩子处)(最先更新的时候，已经是最后一层递归)
        UpdateHeight(root);
        //根据新高度的值来进行旋转
        //都往左插了，只可能能是L型树
        if (BalanceFactor(root) == 2)//说明是L型树
        {
            if (BalanceFactor(root->left) == 1)//说明是LL型树
            {
                //直接右旋即可
                R(root);
            }
            else if (BalanceFactor(root->left) == -1)//说明是LR型树
            {
                //先左旋再右旋
                L(root->left);
                R(root);
            }
        }
    }
    else//如果要插入的值大于当前节点的值
    {
        //那么就要插入该节点的右孩子处
        Insert(root->right, val);
        //更新root的高度(因为是插入了root的左孩子处)(最先更新的时候，已经是最后一层递归)
        UpdateHeight(root);
        //既然往右插了，只可能是R型树
        if (BalanceFactor(root) == -2)//说明是R型树
        {
            if (BalanceFactor(root->right) == -1)//说明是RR型树
            {
                //直接左旋
                L(root);
            }
            else if (BalanceFactor(root->right) == 1)//说明是RL型树
            {
                //先右旋再左旋
                R(root->right);
                L(root);
            }
        }
    }
}

int main()
{
    int n;//节点总数
    cin >> n;
    node* root = nullptr;
    for (int i = 0; i < n; i++)
    {
        int _val;
        cin >> _val;
        Insert(root, _val);
    }
    cout << root->val;
}
```

    这道题目非常具有挑战性。需要对AVL树（二叉平衡树）有足够的了解，要懂得AVL树的插入方法，也就是左旋以及右旋的方法，具体思路在算法笔记已经很详细了，我就不记了，主要说一些我出错的一些点：

* 首先在写的时候我寻思直接调取height不就行了么，何必非得写个`GetHeight`函数呢，写到后面才发现，如果调取height的root是空指针呢？这不就报错了，所以，写成函数的话，可以加一层判断，如果是空指针，那就放回0的高度

* 其次是左旋右旋的问题，其实搞明白了左右旋，但运行时，出现了中断，提示说tmp为空指针，那么为什么tmp是空指针呢？这是因为我函数名写反了，我在推导时，LL型树要进行右旋才可以，但写完右旋函数后，函数命名却是**L**（受LL型树混淆了），所以才会出现空指针的问题

* 最后就是，在跑测试样例时发现第二个不对，看了下，插入时只写了要插入的值小于当前节点值的情况了，把另一个分支给忘了（代码太长了，忘写另一部分了）。此时也正好发现，两种分支各对应一种树也就是R型和L型，这个大类可以率先分开

## A1107 [Social Clusters](https://pintia.cn/problem-sets/994805342720868352/problems/994805361586847744)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
const int maxn = 1010;
vector<int> father(maxn);//记录每个人的父亲节点
vector<int> isRoot(maxn, 0);//记录每个节点是否为根节点，是的话，记录集合有几个元素
vector<int> hobbyFather(maxn, 0);//记录每种hobby的father是谁
int n;//记录节点总数
// 寻找x的根父亲节点 函数
int FindFather(int x)
{
    int a = x;//储存x，用于接下来路径压缩
    //只有x = father[x]时，x才是根节点
    while (x != father[x])
    {
        x = father[x];
    }
    //循环完也就找到x是根父节点
    //路径压缩
    while (a != father[a])
    {
        int z = a;
        a = father[a];//要把a的直接father的父节点也更新
        father[a] = x;//将a也就是一开始的x的父节点重置为根父节点，这样方便下次查询
    }
    return x;//返回父节点
}
//合并两个集合的函数
void Union(int a, int b)
{
    //先找到两个人的根节点
    int fa = FindFather(a);
    int fb = FindFather(b);
    //把其中一个人(fb)的父亲节点设置为对方(fa)
    if(fa != fb)
    father[fb] = fa;
}
//初始化father数组，每个人在成为集合前，自己是自己的father，也就是说随时可以做某个hobby的根父节点
void init()
{
    for (int i = 1; i <= n; i++)
    {
        father[i] = i;
    }
}
bool comp(int a, int b)
{
    return a > b;//大的在前面
}
int main()
{
    //读取节点总数
    cin >> n;
    init();
    for (int i = 1; i <= n; i++)
    {
        int _num;
        cin >> _num;
        getchar();//吸收冒号
        while (_num--)
        {
            int _hobby;
            cin >> _hobby;
            //先检查这个hobby之前有没有人
            if (hobbyFather[_hobby] == 0)//说明没有人
            {
                //那么i就让自己的father自告奋勇，做第一个这个集群的father
                hobbyFather[_hobby] = i;
            }
            //不管有没有人，都将这个i合并到这个爱好的根父节点(之前没有人的话，其实不union也行)
            Union(FindFather(hobbyFather[_hobby]), i);
        }
    }
    //接下来统计每个人的根父节点是谁
    for (int i = 1; i <= n; i++)
    {
        isRoot[FindFather(i)]++;
    }
    int ans = 0;//记录集群总数
    for (int i = 1; i <= n; i++)
    {
        if (isRoot[i] != 0) ans++;
    }
    cout << ans << endl;
    //给集群人数排序
    sort(isRoot.begin() + 1, isRoot.begin() + 1 + n, comp);
    for (int i = 1; i <= ans; i++)
    {
        if (i == 1) cout << isRoot[i];
        else cout << ' ' << isRoot[i];
    }
}
```

    这道题是并查集的一道题，之前从来没做过并查集的题，长见识了，第一次接触还是有点生疏的，注意下面几个点：

* 关于记录根节点，其实只要记录住每个爱好的同好有一个人就行，这样就能直接找到这个爱好所属的集合的根节点，这样的话，都遍历完后，再遍历每个人，去查他的根节点是谁，就可以记录准确的集群人数了
