---
title: PAT-Reversing Linked List & Linked List Sorting & Deduplication on a Linked List & Splitting A Linked List
date: 2022-08-25 16:53:00 +0800
categories: [算法刷题, PAT]
tags: [链表]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1074[Reversing Linked List](https://pintia.cn/problem-sets/994805342720868352/problems/994805394512134144)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
using namespace std;

const int maxn = 100010;
struct node
{
    int v, next;
}L[maxn];
int head, n, k;
int main()
{
    cin >> head >> n >> k;
    for (int i = 0; i < n; i++)
    {
        int addr;
        cin >> addr;
        cin >> L[addr].v >> L[addr].next;
    }
    int tmp = head;
    int preLast = -1, pre, cur;
    while (tmp != -1)
    {
        int tmpK = 1,tmpH = tmp;
        while (tmpH != -1 && tmpK < k)
        {
            tmpH = L[tmpH].next;
            tmpK++;
        }
        if (tmpK < k)
        {
            if(preLast != -1)
                L[preLast].next = tmp;
            break;
        }
        cur = L[tmp].next;
        pre = tmp;
        tmpK -= 1;
        while (tmpK--)
        {
            int next = L[cur].next;
            L[cur].next = pre;
            pre = cur;
            cur = next;
        }
        if (tmp == head) head = pre;//记录反转后的头节点
        else L[preLast].next = pre;
        preLast = tmp;
        L[preLast].next = -1;//每次翻转完后，先设为-1，如果后面还有数据，就会更新的
        tmp = cur;
    }
    while (head != -1)
    {
        if (L[head].next != -1)
            printf("%05d %d %05d\n", head, L[head].v, L[head].next);
        else
            printf("%05d %d %d\n", head, L[head].v, L[head].next);

        head = L[head].next;
    }
}
```

    首先，这是我写的实打实地去翻转链表，但花了很久时间去debug，最开始32分钟只拿了17/25分，后面debug了很久拿到了22分，依旧有一个点过不去，但是在牛客上可以完美通过，麻了。

    其实真的去翻转还挺麻烦的，需要注意的点太多了，还不如昨天用哈希表呢，虽然慢但好歹拿满分了，本来想着今天就纯用一次静态链表试试，没想到结果竟是如此，说说这思路需要注意的点吧：

* 这道题需要设置很多很多变量来存地址，其中prelast就是上一次翻转完的最后一个节点，因为要和后面的一段连接上，并且else L[preLast]每次都要要初始化为-1，这样如果后面没有节点了，就正好了。

* 要注意变量更改的位置，不能提前更改完了，后面又要用这个变量了，这样就出错了，得先存起来。

    好了，不说这个方法了，还是说说普遍的方法吧，就是要把所有有效的节点存起来，然后去翻转每一段的地址的值就行了。

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<algorithm>
using namespace std;

const int maxn = 100010;
struct node
{
    int v, next;
}L[maxn];
int head, n, k;
int main()
{
    cin >> head >> n >> k;
    for (int i = 0; i < n; i++)
    {
        int addr;
        cin >> addr;
        cin >> L[addr].v >> L[addr].next;
    }
    int head2 = head;
    int list[maxn];//存地址
    int sum = 0;
    while (head2 != -1)
    {
        list[sum++] = head2;
        head2 = L[head2].next;
    }
    for (int i = 0; i < (sum - sum % k); i+=k)
    {
        reverse(list + i, list + i + k);
    }
    for (int i = 0; i < sum - 1; i++)
    {
        printf("%05d %d %05d\n", list[i], L[list[i]].v, list[i + 1]);
    }
    printf("%05d %d -1\n", list[sum - 1], L[list[sum - 1]].v);
}
```

    这样写就很简单，很迅速，我咋就想不到呢。注意的点是：在翻转的时候，应该小于`(sum - sum % k)`而不是`(n - n % k)`，因为里面可能有无效的数据，我一开始写的n竟然还有24分，怪。

## A1052[Linked List Sorting](https://pintia.cn/problem-sets/994805342720868352/problems/994805425780670464)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<algorithm>
using namespace std;

const int maxn = 100010;
struct node
{
    int v, next;
}L[maxn];
bool comp(int a, int b)
{
    return L[a].v < L[b].v;
}
int list[maxn];
int main()
{
    int head, n;
    cin >> n >> head;
    for (int i = 0; i < n; i++)
    {
        int addr;
        cin >> addr;
        cin >> L[addr].v >> L[addr].next;
    }
    int sum = 0;
    while (head != -1)
    {
        list[sum++] = head;
        head = L[head].next;
    }
    if (sum == 0)
    {
        printf("0 -1");
        return 0;
    }
    sort(list, list + sum, comp);
    printf("%d %05d\n", sum, list[0]);
    for (int i = 0; i < sum - 1; i++)
    {
        printf("%05d %d %05d\n", list[i], L[list[i]].v, list[i + 1]);
    }
    printf("%05d %d -1\n", list[sum - 1], L[list[sum - 1]].v);
}
```

    13分钟拿下24/25分，多亏了前面那道题的经验。那么这最后一份怎么回事呢，我猜是sum=0的时候，但我就打印了0，所以还是过不去，去牛客看了眼，得打印0 -1，才对，麻了。

## A1097 [Deduplication on a Linked List](https://pintia.cn/problem-sets/994805342720868352/problems/994805369774129152)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<unordered_map>
#include<algorithm>
using namespace std;

const int maxn = 100010;
struct node
{
    int v, next;
}L[maxn];
vector<int> List,List2;
unordered_map<int, int> map;
int main()
{
    int head, n;
    cin >> head >> n;
    for (int i = 0; i < n; i++)
    {
        int addr;
        cin >> addr;
        cin >> L[addr].v >> L[addr].next;
    }

    while (head != -1)
    {
        if (map.count(abs(L[head].v)))
        {
            List2.emplace_back(head);
        }
        else
        {
            map[abs(L[head].v)]++;
            List.emplace_back(head);
        }
        head = L[head].next;
    }
    for (int i = 0; i < List.size(); i++)
    {
        if(i != List.size() - 1)
            printf("%05d %d %05d\n", List[i], L[List[i]].v, List[i + 1]);
        else 
            printf("%05d %d -1\n", List[i], L[List[i]].v);
    }

    for (int i = 0; i < List2.size(); i++)
    {
        if(i != List2.size() - 1)
            printf("%05d %d %05d\n", List2[i], L[List2[i]].v, List2[i + 1]);
        else
            printf("%05d %d -1\n", List2[i], L[List2[i]].v);
    }

}
```

    15分钟23/25分，19分钟拿满，中间有个段错误，这是因为我之前在打印的时候，会单独把表的最后一个打印，直接用了`List.back()`，但这个List有可能是空的，自然就会触发段错误。

## A1133 [Splitting A Linked List](https://pintia.cn/problem-sets/994805342720868352/problems/994805346776760320)

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<unordered_map>
#include<algorithm>
using namespace std;

const int maxn = 100010;
struct node
{
    int v, next;
}L[maxn];

vector<int> list1, list2, list3;

int main()
{
    int head, n, k;
    cin >> head >> n >> k;
    for (int i = 0; i < n; i++)
    {
        int addr;
        cin >> addr;
        cin >> L[addr].v >> L[addr].next;
    }

    while (head != -1)
    {
        if (L[head].v < 0)
        {
            list1.emplace_back(head);
        }
        else if (L[head].v <= k)
        {
            list2.emplace_back(head);
        }
        else
        {
            list3.emplace_back(head);
        }
        head = L[head].next;
    }
    for (int i = 0; i < list2.size(); i++)
    {
        list1.emplace_back(list2[i]);
    }
    for (int i = 0; i < list3.size(); i++)
    {
        list1.emplace_back(list3[i]);
    }
    for (int i = 0; i < list1.size(); i++)
    {
        if (i != list1.size() - 1)
            printf("%05d %d %05d\n", list1[i], L[list1[i]], list1[i + 1]);
        else
            printf("%05d %d -1\n", list1[i], L[list1[i]]);
    }
}
```

    8分17秒拿下，pat的链表题属于是套路题了，记得换行，遍历链表的时候记得更新指针为next，不然会内存超限。
