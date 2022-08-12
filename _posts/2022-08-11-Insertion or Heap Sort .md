---
title: PAT-Insertion or Heap Sort & 
date: 2022-08-11 23:38:00 +0800
categories: [算法刷题, PAT]
tags: [堆排序]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## 1098 [Insertion or Heap Sort](https://pintia.cn/problem-sets/994805342720868352/problems/994805368847187968)

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

const int maxn = 101;
int n;//元素个数
vector<int> origin(maxn);//原始数组
vector<int> partial(maxn);//部分排序的数组

bool IsSame(vector<int>& tmp)//比较当前排序的序列是否和partial一致
{
	for (int i = 1; i <= n; i++)
	{
		if (tmp[i] != partial[i]) return false;
	}
	return true;
}
void ShowArray(vector<int> a)
{
	for (int i = 1; i <= n; i++)
	{
		if (i == 1) cout << a[i];
		else cout << ' ' << a[i];
	}
}
//插入排序
bool Insert(vector<int>& origin)
{
	bool flag = false;//用来判断是否和partial一致
	for (int i = 2; i <= n; i++)//进行n-1次插入
	{
		if (i != 2 && IsSame(origin))//第一次插入前不进行判断
		{
			flag = true;
		}
		//开始插入
		int j = i;
		while (j > 1 && origin[j - 1] > origin[j])//如果j前一位大于j
		{
			int tmp = origin[j - 1];
			origin[j - 1] = origin[j];
			origin[j] = tmp;
			j--;
		}
		//判断是否是否和partial一样，一样的话就终止循环，返回true了
		if (flag) return true;
	}
	//排完了，也没和partial一样
	return false;
}
//向下调整函数
void DownAdjust(int low, int high)
{
	int i = low, j = i * 2;//i为欲调整节点，j为孩子节点
	while (j <= high)//还有孩子节点
	{
		//判断是否有右孩子,如果有右孩子而且比左孩子大
		if (j + 1 <= high && origin[j + 1] > origin[j])
		{
			j = j + 1;//记录最大的孩子为右孩子
		}
		//判断i与j的大小
		if (origin[i] < origin[j])//如果欲调整节点小于其最大的孩子节点
		{
			//那就要开始向下走了
			swap(origin[i], origin[j]);//交换两个节点的值
			i = j;//i移动到j了，要继续向下调整，直到调整到合适的位置
			j = i * 2;//j还是i的孩子节点
		}
		else break;//如果调整到合适位置了，就退出
	}
}

//堆排序
void Heap()
{
	bool flag = false;//判断是否已经和partial一致了
	//先建堆
	for (int i = n / 2; i >= 1; i--)//只用向下调整每个非叶子节点（一共有n/2个非叶子节点）
	{
		DownAdjust(i, n);
	}
	//开始堆排序
	for (int i = n; i > 1; i--)//排n-1次
	{
		//先判断是否和partial一样了
		if (i != n && IsSame(origin))
		{
			flag = true;
		}
		swap(origin[1], origin[i]);//堆顶元素（最大的）放到序列尾部
		//堆排序
		DownAdjust(1, i - 1);//要把i排除掉，因为i往后都是排好序的了
		if (flag)
		{
			ShowArray(origin);
			break;
		}
	}
}


int main()
{
	cin >> n;
	for (int i = 1; i <= n; i++)
	{
		cin >> origin[i];
	}
	for (int i = 1; i <= n; i++)
	{
		cin >> partial[i];
	}
	vector<int> temp(origin);
	if (Insert(temp))
	{
		cout << "Insertion Sort" << endl;
		ShowArray(temp);
	}
	else//说明是堆排序
	{
		cout << "Heap Sort" << endl;
		Heap();
	}

}
```

    这道题涉及到了堆排序，堆排序主要就是向下调整函数，需要注意的点：

* 堆的构建直接用数组，并且起始下标从1开始，这样方便之后计算其孩子节点的下标，即i*2，所以插入排序也要从下标1开始

* 创建堆时，只需要向下调整非叶子节点，对于完全二叉树而言，非叶子节点的个数为n/2个

* 堆排序时，需要调换数组首尾的值，并且接着做向下调整来维持堆，注意此时向下调整的high要为i-1，因为i及往后都已经是排好序的了

* 堆排序是大顶堆

* 需要注意的一点是，刚开始还没有排的时候，不要判断是否和partial一致，否则会出现歧义


