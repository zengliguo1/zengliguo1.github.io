---
title: PAT-Integer Factorization & Acute Stroke & Tree Traversals Again & Invert a Binary Tree & Total Sales of Supply Chain 
date: 2022-08-08 21:54:00 +0800
categories: [算法刷题, PAT]
tags: [dfs, bfs, 二叉树的遍历, 树的遍历]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1103 [Integer Factorization](https://pintia.cn/problem-sets/994805342720868352/problems/994805364711604224)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<cmath>
#include<cstdio>
using namespace std;
vector<int> fac, ans, tmp;//fac储存小于n的i^p数组，ans储存最后的底，tmp储存临时的底
int n = -1, k = -1, p = -1;
int maxSum = -1;
void Fac()
{
    for (int i = 0; i <= n; i++)
    {
        int power = pow(i, p);
        if (power > n) break;
        else
        {
            fac.emplace_back(power);
        }
    }
}
//回溯三部曲
//1.确认返回类型和参数
//当前回溯的fac数组下标index，目前和sum，当前数字个数curK，当前底的和facSum
void dfs(int index, int sum, int curK, int facSum)
{
    //2.中止条件
    if (curK == k && sum == n)//当数字个数满足条件k，而且当前和已经相等了
    {
        //先判断是否要更新当前的答案数组
        if (facSum > maxSum)
        {
            maxSum = facSum;//更新最大和
            ans = tmp;//更新答案数组
        }
        return;
    }
    //如果当前数字个数到k了，但总和却不是n；或者当前总和大于等于n，但数字个数不到k
    if (curK > k || sum > n ) return;
    //3.每层回溯的遍历过程
    for (int i = index; i > 0; i--)
    {
        tmp.emplace_back(i);
        dfs(i, sum + fac[i], curK + 1, facSum + i);
        tmp.pop_back();
    }

}
int main()
{
    //cin >> n >> k >> p;
    scanf("%d %d %d", &n, &k, &p);
    //先算出i^p小于n的数组
    Fac();
    //回溯,从最大下标开始算
    dfs(fac.size() - 1, 0, 0, 0);
    //打印结果：
    if (ans.empty())
    {
        cout << "Impossible";
    }
    else
    {
        cout << n << " =";
        for (int i = 0; i < ans.size(); i++)
        {
            if (i != ans.size() - 1)//如果不是最后一项
                printf(" %d^%d +", ans[i], p);
                //cout << ' ' << ans[i] << '^' << p << " +";
            else
                printf(" %d^%d", ans[i], p);
                //cout << ' ' << ans[i] << '^' << p;
        }
    }
}
```

    许久不做回溯的题目，这道题都有点不会了。主要是开始不知道该遍历哪些数，看了算法笔记，提前把需要遍历的数存到fac数组中去，有几个点需要注意下：

* 开始输出不对，检查后发现是在插入tmp中时，应该插入i而不是index

* 后来有一处一直超时，把cin和cout全改为printf和scanf还是不行；接着又改了下遍历部分，其实i是不会=0的，还是不行；又改了下剪枝，`if (curK > k || sum > n ) return;`，之前此处是个比较复杂的判断：`if((curK == k && sum != n) || sum > n)`，这样会超时。

## A1091[Acute Stroke](https://pintia.cn/problem-sets/994805342720868352/problems/994805375457411072)

```cpp
#include<iostream>
#include<vector>
#include<queue>
using namespace std;
//节点结构体
struct node
{
    int x, y, z;
}Node;
int m, n, l, t;//m*n矩阵，l切片层数, t阈值
int ans = 0;//最终结果
vector< vector< vector<int> > > brain(1290, vector<vector<int>>(130, vector<int>(61)));
//int brain[1290][130][61];
bool inque[1290][130][61] = {false};

//增量数组
int xOff[6] = { 0, 0, 0, 0, 1, -1 };
int yOff[6] = { 0, 0, 1, -1, 0, 0 };
int zOff[6] = { 1, -1, 0, 0, 0, 0 };

//判断是否应该入队
bool valid(int x, int y, int z)
{
    //越界返回false
    if (x < 0 || x >= m || y < 0 || y >= n || z < 0 || z >= l) return false;
    //如果之前入过队或者是0，也返回false
    if (inque[x][y][z] || brain[x][y][z] == 0) return false;
    //否则就可以入队
    return true;
}
//返回这一区域的所有为1的个数
int bfs(int x, int y, int z)
{
    int total = 0;//统计当前块的所有1的数量
    queue<node> q;//储存坐标的队列
    Node.x = x;
    Node.y = y;
    Node.z = z;
    //先设置为入过队了
    inque[x][y][z] = true;
    q.push(Node);
    //只要队列不空，就遍历队列中的每个元素，去查它上下前后左右6个元素是否连接
    while (!q.empty())
    {
        node top = q.front();//获取队首
        q.pop();//出队
        //总数++
        total++;
        for (int i = 0; i < 6; i++)
        {
            int newX = top.x + xOff[i];
            int newY = top.y + yOff[i];
            int newZ = top.z + zOff[i];
            if (valid(newX, newY, newZ))//如果该节点有效，入队
            {
                inque[newX][newY][newZ] = true;//设置该坐标为入过队了
                Node.x = newX, Node.y = newY, Node.z = newZ;
                q.push(Node);//入队
            }
        }
    }
    if (total >= t) return total;
    else return 0;
}

int main()
{
    cin >> m >> n >> l >> t;
    for (int z = 0; z < l; z++)//z层切片
    {
        for (int x = 0; x < m; x++)//x行矩阵
        {
            for (int y = 0; y < n; y++)//y列矩阵
            {
                cin>>brain[x][y][z];
            }
        }
    }
    for (int z = 0; z < l; z++)//z层切片
    {
        for (int x = 0; x < m; x++)//x行矩阵
        {
            for (int y = 0; y < n; y++)//y列矩阵
            {        
                if (brain[x][y][z] == 1 && !inque[x][y][z])
                {
                    ans += bfs(x, y, z);
                }
            }
        }
    }
    cout << ans;
}
```

    这道题目还是有一定难度的，需要注意的点比较多，接下来一一列出：

* 首先，也是最最需要注意的一点就是x,y,z这三个坐标所对应的三位数组的x,y,z，比如，在这道题目中x,y,z分别对应的是行，列和层，其实顺序可以颠倒，只要对应关系都保持一致即可，不然的话会内存超限，卡在循环里出不来。比如我开始是拿y做行，但是呢，我申请的数组y确实列的大小，在设置inque数组时x,y,z的对应也出了问题，导致卡在循环里出不来

* 其次，申请数组时，可以根据需求申请，其实直接申请数组，比申请vector要方便的多

* 注意total变量自加的区域，不要重复添加

* 需要写三个增量数组，这样的话方便搜索时循环遍历

## A1086 [Tree Traversals Again](https://pintia.cn/problem-sets/994805342720868352/problems/994805380754817024)

```cpp
#include<iostream>
#include<vector>
using namespace std;
//节点结构体
struct node
{
    int val;
    node* left;
    node* right;
    node(int _val) :val(_val), left(nullptr), right(nullptr) {}
}*root;
int n;//节点个数
vector<int> pre, in, post;
node* build(int preL, int preR, int inL, int inR)
{
    if (preL > preR) return nullptr;
    //创建新的节点
    node* n = new node(pre[preL]);
    //查找val在中序遍历数组中的下标
    int index;
    for (int i = 0; i < in.size(); i++)
    {
        if (in[i] == pre[preL]) index = i;
    }
    //构建该节点的左右子树
    n->left = build(preL + 1, index - inL + preL, inL, index - 1);
    n->right = build(preR - inR + index + 1, preR, index + 1, inR);
    return n;
}
//后序遍历
void travelsal(node* root)
{
    //中止条件
    if (root == nullptr) return;
    //后序遍历
    travelsal(root->left);
    travelsal(root->right);
    post.emplace_back(root->val);
}
int main()
{
    cin >> n;
    vector<int> stack;
    for (int i = 0; i < 2*n; i++)
    {
        string str;
        cin >> str;
        if (str == "Push")
        {
            int _pre;
            cin >> _pre;
            pre.emplace_back(_pre);
            stack.emplace_back(_pre);//入栈
        }
        else
        {
            in.emplace_back(stack.back());//获取栈顶元素，放入中序遍历数组中
            stack.pop_back();//出栈
        }
    }
    //通过前序遍历数组和中序遍历数组构造二叉树，再得到后序遍历数组
    root = build(0, n - 1, 0, n - 1);
    //后序遍历
    travelsal(root);
    //打印结果
    for (int i = 0; i < n; i++)
    {
        if (i == 0)
            cout << post[i];
        else
            cout << ' ' << post[i];
    }
}
```

    这道题经典通过两序推另一序，没啥好说的。

## A1102 [Invert a Binary Tree](https://pintia.cn/problem-sets/994805342720868352/problems/994805365537882112)

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<string>
using namespace std;
//节点结构体
struct node
{
    int left = -1;
    int right = -1;
};
vector<node> tree;
vector<bool> isRoot(11, true);
int n;//节点个数
vector<int> level, in;
void levelTravelsal(int root)
{
    queue<int> q;
    q.push(root);
    while (!q.empty())
    {
        int top = q.front();
        level.emplace_back(top);
        q.pop();
        //查top的左右子树，并加入队列
        if (tree[top].left != -1) q.push(tree[top].left);
        if (tree[top].right != -1) q.push(tree[top].right);
    }
}
void inTravelsal(int root)
{
    if (root == -1) return;
    inTravelsal(tree[root].left);
    in.emplace_back(root);
    inTravelsal(tree[root].right);
}
int main()
{
    cin >> n;
    //读取的时候顺便也就翻转了
    for (int i = 0; i < n; i++)
    {
        node n;
        string l, r;
        cin >> r >> l;
        if (l != "-")//说明不是空节点
        {
            n.left = stoi(l);
            isRoot[n.left] = false;//该节点不可能是根节点
        }
        if (r != "-")
        {
            n.right = stoi(r);
            isRoot[n.right] = false;
        }
        tree.emplace_back(n);
    }
    //寻找根节点下标
    int root;
    for (int i = 0; i < n; i++)
    {
        if (isRoot[i]) root = i;
    }
    //层序遍历
    levelTravelsal(root);
    for (int i = 0; i < n; i++)
    {
        if (i == 0)
            cout << level[i];
        else
            cout << ' ' << level[i];
    }
    cout << endl;
    //中序遍历
    inTravelsal(root);
    for (int i = 0; i < n; i++)
    {
        if (i == 0)
            cout << in[i];
        else
            cout << ' ' << in[i];
    }
}
```

    这道题目不是很难，用到了静态树，没有用指针。需要注意的是，得记录根节点的下标。

## A1079 [Total Sales of Supply Chain](https://pintia.cn/problem-sets/994805342720868352/problems/994805388447170560)

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<cstdio>
#include<cmath>
using namespace std;
//节点结构体
struct node
{
    double weight;//货物量
    vector<int> child;
};
int n;//节点总数
double price, r;//单价和涨幅
double total = 0.0;
vector<node> supply;
//回溯三部曲
//1.确认返回类型和参数
void travelsal(int root, int depth)
{
    //2.终止条件
    //先判断是不是叶子节点
    if (supply[root].child.empty())//是叶子节点，算钱,中止
    {
        total += (supply[root].weight * price * pow(1 + r, depth));
    }
    else//不是叶子节点
    {
        //3.每个回溯的遍历过程
        for (int c : supply[root].child)
        {
            travelsal(c, depth + 1);
        }
    }
}

int main()
{
    cin >> n >> price >> r;
    r /= 100;
    for (int i = 0; i < n; i++)
    {
        int _num;
        cin >> _num;
        node nd;
        if (_num == 0)//说明是零售商
        {

            cin>>nd.weight;//读取需求量
        }
        else//不是零售商
        {    
            for (int j = 0; j < _num; j++)//记录每一个孩子
            {
                int _child;
                cin >> _child;
                nd.child.emplace_back(_child);
            }
        }
        supply.emplace_back(nd);//将该节点加到静态树中
    }

    travelsal(0, 0);
    printf("%.1f", total);
}
```

    这道题目呢，其实就是一个多叉树的遍历，顺便记录下深度，需要注意的就是读入的r要除以一百。
