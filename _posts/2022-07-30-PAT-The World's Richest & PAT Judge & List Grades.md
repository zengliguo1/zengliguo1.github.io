---
title: PAT-The World's Richest & PAT Judge & List Grades
date: 2022-07-30 22:56:00 +0800
categories: [算法刷题, PAT]
tags: [排序]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1055 [The World's Richest](https://pintia.cn/problem-sets/994805342720868352/problems/994805421066272768)

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
using namespace std;

const int maxn = 100001;//最大人数
int Age[201] = {0};//某个年龄的人数
struct personInfo
{
    string name;
    int age;
    int netWorth;
    personInfo(){}
    personInfo(string _name, int _age, int _netWorth): name(_name), age(_age), netWorth(_netWorth){}
}people[maxn];
bool comp(personInfo &p1, personInfo &p2)
{
    if(p1.netWorth != p2.netWorth)
    {
        return p1.netWorth > p2.netWorth;
    }
    else if(p1.age != p2.age)
    {
        return p1.age < p2.age;
    }
    else
    {
        return p1.name < p2.name;
    }
}
int main()
{
    //读入数据
    int N,K;//总人数、询问数
    cin>>N>>K;
    for(int i = 0; i < N; i++)
    {
        string _name;
        int _age, _netWorth;
        cin>>_name>>_age>>_netWorth;
        people[i] = personInfo(_name, _age, _netWorth);
    }
    //先排序
    sort(people, people + N, comp);
    //从排序好的所有人中，再进行筛选，每个年龄的人数最多为100个
    vector<personInfo> valid;
    for(int i = 0; i < N; i++)
    {
        if(Age[people[i].age] < 100)//如果当前年龄的人数小于100
        {
            valid.emplace_back(people[i]);
            Age[people[i].age]++;
        }
    }
    //处理询问
    for(int j = 1; j <= K; j++)
    {
        int M, Amin, Amax;
        cin>>M>>Amin>>Amax;
        vector<personInfo> output;
        for(int i = 0; i < valid.size(); i++)
        {
            //在年龄区间的话就加入output中
            if(valid[i].age >= Amin && valid[i].age <= Amax)
            {
                output.emplace_back(valid[i]);
            }
            //够数了就退出循环，否则找到底
            if(output.size() == M) break;
        }
        //输出：
        cout<<"Case #"<<j<<":"<<endl;
        if(output.size() == 0)//没有符合要求的人
        {
            cout<<"None"<<endl;
        }
        else
        {
            for(auto p : output)
            {
                cout<<p.name<<' '<<p.age<<' '<<p.netWorth<<endl;
            }
        }
    }
}
```

    这道题其实不难，但我开始的思路是不对的，我本来是想在每次的询问中排序搜索，但这样是不行的，会超时，所以要提前排好序，再按顺序去查，但即使这样，也会超时，所以就要再进行一次筛选，把每个年龄的人数控制在100以内，这样就又剩下很多时间，多亏算法笔记，不然不知道又要在这耗费多少时间。

    还有一处小问题就是，构造函数初始化，要以`variable(value)`这样的形式来赋值，不能用`=`不然会报错。

## A1075 [PAT Judge](https://pintia.cn/problem-sets/994805342720868352/problems/994805393241260032)

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
#include<cstdio>
using namespace std;

const int maxn = 10001;
struct stuInfo
{
    int id;
    int point[6] = { -1 , -1, -1 , -1 , -1 , -1 };
    int total = 0;
    int perfectNum = 0;
    bool show = 0;
}stu[maxn];
bool comp(stuInfo &s1, stuInfo &s2)
{
    if (s1.total != s2.total) return s1.total > s2.total;
    else if (s1.perfectNum != s2.perfectNum) return s1.perfectNum > s2.perfectNum;
    else if (s1.show != s2.show) return s1.show > s2.show;//如果两个人都是0分，那么show的要排在不show的前面
    else return s1.id < s2.id;
}
int main()
{

    int N, K, M;//总人数、题目数、提交数
    cin >> N >> K >> M;
    int p[6];//每道题的满分
    for (int i = 1; i <= K; i++) cin >> p[i];
    //读取每个提交信息
    while (M--)
    {
        int _id, _quest, _point;
        cin >> _id >> _quest >> _point;
        stu[_id].id = _id;//刚开始每个人的id就是输入的_id
        if (_point != -1)//如果提交编译通过了
        {
            //保存更新前的分数
            int tmp = stu[_id].point[_quest];
            if (_point <= tmp) continue;//如果这次得分小于等于上次(正好如果多次满分也只+一次完美题目数)
            //如果这次分高，就更新
            stu[_id].point[_quest] = _point > tmp ? _point : tmp;
            stu[_id].show = 1;//需要显示
            if (_point == p[_quest]) stu[_id].perfectNum++;//这道题目满分
            if (tmp == -1)//说明之前这道题没提交成功过，这次提交成功了
            {
                stu[_id].total += _point;//更新总分
            }
            else//以前有分数，那就更新为最高分数
            {
                //当前分数-上次分数
                stu[_id].total += (stu[_id].point[_quest] - tmp);
            }
        }
        else//提交了但没通过
        {
            //虽然不显示，但-1要变成0(如果之前有分数，那自然不用改)
            if(stu[_id].point[_quest] == -1) stu[_id].point[_quest] = 0;
        }
    }
    //排序
    sort(stu + 1, stu + N + 1, comp);
    //打印
    int rank = 0;//从0开始,但有个判断会先+1
    for (int i = 1; i <= N; i++)
    {
        if (stu[i].show)//首先得显示
        {
            //如果当前人的分数和上一个一样，那排名不变
            if (i > 1 && stu[i].total == stu[i - 1].total)
            {
            }
            else
            {
                rank = i ;
            }
            printf("%d %05d %d ", rank, stu[i].id, stu[i].total);
            //打印每个题目的分数
            for (int j = 1; j <= K; j++)
            {
                if (j != 1) cout << " ";
                if (stu[i].point[j] == -1)//说明没提交成功
                {
                    cout << "-";
                }
                else cout << stu[i].point[j];
            }
            cout << endl;
        }
    }
}
```

    这道题，坑挺多的，我还都踩了一遍......

* 首先最最基础的问题就是，下标问题，输入的下标都是从1开始的，我也是按1算的，但循环我都从0开始了，所以一开始导致出了很多顺序错乱的问题，同时，既然下标从1开始，那么申请数组的时候也要多申请一个

* 第二个问题，数组初始化为-1时，不能简单地`int a[5] = { -1 }`，这样的话，只有第一个元素会被赋值成-1，后面的都是0，好在我的`point`数组只有6个，可以直接6个-1，不然的话就用`memset(point, -1, sizeof(point))`。

* 还有问题就是，如果一个人多次提交满分，但完美解决问题数不能多加，这个只需要加上`if (_point <= tmp) continue;`即可解决

* 最后的问题，就是如果一个人提交了，但没通过，那么输出时，就不能是负号，应该是0，我在加了判断后，确实可以表示0了，但有个问题，就是如果一个人他已经有分数了，然后又提交时没有通过，这时候不能用0覆盖掉原有的分数，加个判断即可：`if(stu[_id].point[_quest] == -1) stu[_id].point[_quest] = 0;`

## A1083 [List Grades](https://pintia.cn/problem-sets/994805342720868352/problems/994805383929905152)

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
using namespace std;

struct stuInfo
{
    string name;
    string id;
    int grade;
    stuInfo(string _name, string _id, int _grade) : name(_name), id(_id), grade(_grade) {}
};

bool comp(stuInfo& s1, stuInfo& s2)
{
    return s1.grade > s2.grade;
}
int main()
{
    int N;//学生数
    cin >> N;
    vector<stuInfo> stu;
    //读取数据
    while (N--)
    {
        string _name, _id;
        int _grade;
        cin >> _name >> _id >> _grade;
        stu.emplace_back(stuInfo(_name, _id, _grade));
    }
    //排序
    sort(stu.begin(), stu.end(), comp);
    //筛选
    int grade1, grade2;
    int cnt = 0;
    cin >> grade1 >> grade2;
    for (auto s : stu)
    {
        if (s.grade >= grade1 && s.grade <= grade2)
        {
            cout << s.name << ' ' << s.id << endl;
            cnt++;
        }
    }
    if (cnt == 0) cout << "NONE";
}
```

    这道题so ez，但即使so ez也没一遍过，先是报错，忘记读取N了，接着是多打印了成绩，只用打印人的姓名和id即可。
