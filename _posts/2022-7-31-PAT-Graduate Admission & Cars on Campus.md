---
title: PAT-Graduate Admission & Cars on Campus
date: 2022-07-31 16:59:00 +0800
categories: [算法刷题, PAT]
tags: [排序, 哈希表]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1080 [Graduate Admission](https://pintia.cn/problem-sets/994805342720868352/problems/994805387268571136)

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
#include<set>
using namespace std;

struct stuInfo
{
    int id;//学生编号
    int G_E, G_I;//统考分数和面试分数
    int total;//学生总分
    int school = -1;//录取学校
    vector<int> prefer;//目标院校
    stuInfo(int _id, int _Ge, int _Gi) :id(_id), G_E(_Ge), G_I(_Gi), total(_Ge + _Gi) {}
};
bool comp(stuInfo& s1, stuInfo& s2)
{
    if (s1.total != s2.total) return s1.total > s2.total;
    else return s1.G_E > s2.G_E;
}

int main()
{
    int N, M, K;//考生数目，学校数，志愿数
    cin >> N >> M >> K;
    vector<int> quota;//学校配额
    for (int i = 0; i < M; i++)
    {
        int _quota;
        cin >> _quota;
        quota.emplace_back(_quota);
    }
    //读取学生信息
    vector<stuInfo> stu;
    for (int i = 0; i < N; i++)
    {
        int _Ge, _Gi;
        cin >> _Ge >> _Gi;
        //存id、成绩
        stu.emplace_back(stuInfo(i, _Ge, _Gi));
        //存志愿
        for (int j = 0; j < K; j++)
        {
            int _prefer;
            cin >> _prefer;
            stu[i].prefer.emplace_back(_prefer);
        }
    }
    //排序：按总分排，总分一样，按统考分
    sort(stu.begin(), stu.end(), comp);
    //学校录取的人
    vector<set<int> > schoolAdmit(M);
    int index = 0;//记录相同排名的学生的第一个下标
    //开始遍历学生，投递学校
    for (int i = 0; i < N; i++)
    {
        //判断当前学生跟上一名学生的名次是否相同，相同的话，index不变，如果不相同，index更新
        if(i > 0 && stu[i].total == stu[i - 1].total && stu[i].G_E == stu[i - 1].G_E)
        {
        }
        else//不同的话
        {
            index = i;//更新下标
        }
        //开始遍历学生的志愿，如果有配额就入，
        //遇到配额不足的，就查相同名次的学生，看其是否在这所学校，如果在，那就也入
        for (int j = 0; j < K; j++)
        {
            int _prefer = stu[i].prefer[j];//学生当前志愿学校
            if (quota[_prefer] > 0)//如果有配额
            {
                stu[i].school = _prefer;
                schoolAdmit[_prefer].insert(stu[i].id);//当前学校录取当前学生
                quota[_prefer]--;//配额减1
                break;//有学上了，就不再看下一个志愿了
            }
            else//如果没有配额
            {
                bool admitted = false;//判断是否录取
                //检查相同名次学生的录取院校是否是当前院校
                for (int k = index; k < i; k++)
                {
                    if (stu[k].school == _prefer)//如果相同名次的学生已经入学
                    {
                        //该生也要入学
                        stu[i].school = _prefer;
                        schoolAdmit[_prefer].insert(stu[i].id);//当前学校录取当前学生，记录编号
                        quota[_prefer]--;//配额减1
                        admitted = true;
                        break;
                    }
                }//循环完都没有这种情况，那就看下一个志愿
                if (admitted) break;//有学上就不再看下一个志愿了
            }
        }
    }
    //打印结果
    for (int i = 0; i < M; i++)
    {
        if (schoolAdmit[i].size() == 0)//如果这所学校没有录取到学生
        {
            cout << endl;
        }
        else
        {
            for (auto j = schoolAdmit[i].begin(); j != schoolAdmit[i].end(); j++)//打印每个学生的id
            {
                if (j != schoolAdmit[i].begin())//如果不是第一个，就前面输出空格
                {
                    cout << ' ' << *j;
                }
                else cout << *j;//是第一个前面就不输出空格
            }
            cout << endl;
        }
    }

}
```

    这道题目看起来代码多，其实也就逻辑复杂点，并不算难，写完后有两个错，一个是段错误，这是因为在遍历学生时，取了`<=N`，但学生是从0开始算的，所以不用`=`。第二个错是学校存的id有误，这是因为要插入的是学生的编号，不是排名。

## A1095 [Cars on Campus](https://pintia.cn/problem-sets/994805342720868352/problems/994805371602845696)

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
#include<cstdio>
#include<map>
using namespace std;

struct carRecord
{
    string plate;
    int time;//转换成s
    string state;//记录车的状态
    carRecord(string _plate, int _time, string _state) :plate(_plate), time(_time), state(_state) {}
};
bool comp(carRecord& r1, carRecord& r2)
{
    if (r1.plate != r2.plate) return r1.plate < r2.plate;
    else return r1.time < r2.time;
}
bool comp2(carRecord& r1, carRecord& r2)
{
    return r1.time < r2.time;
}
int main()
{
    int N, K;//N个记录，K个询问
    cin >> N >> K;
    vector<carRecord> record;
    //读取记录
    while (N--)
    {
        string _plate, _state;
        int hh, mm, ss;
        //scanf读取string的方法：
        _plate.resize(8);
        _state.resize(4);
        scanf("%s %d:%d:%d %s", &_plate[0], &hh, &mm, &ss, &_state[0]);
        record.emplace_back(carRecord(_plate, hh * 3600 + mm * 60 + ss, _state));
    }
    //排序
    sort(record.begin(), record.end(), comp);
    //提取有效数据
    vector<carRecord> valid;
    map<string, int> totalTime;//记录每辆车的总时间
    int maxTime = 0;
    string in = "in";
    string out = "out";
    in.resize(4);
    out.resize(4);
    for (int i = 0; i < record.size() - 1; i++)
    {
        //检查相邻两个是不是能够配对
        if (record[i].plate == record[i + 1].plate)//如果相邻两个记录车牌号一样
        {
            //检查是否一进一出
            string r1 = record[i].state;
            string r2 = record[i + 1].state;
            if (r1 == in && r2 == out)
            {
                valid.emplace_back(record[i]);
                valid.emplace_back(record[i + 1]);
                totalTime[record[i].plate] += (record[i + 1].time - record[i].time);
                if (totalTime[record[i].plate] > maxTime) maxTime = totalTime[record[i].plate];//记录最久时间
            }
        }
    }
    //把valid按时间排序
    sort(valid.begin(), valid.end(), comp2);
    //开始处理询问
    int now = 0, numCar = 0;//now指当前询问的车辆序号
    for (int i = 0; i < K; i++)
    {
        int hh, mm, ss;
        scanf("%d:%d:%d", &hh, &mm, &ss);
        int curTime = hh * 3600 + mm * 60 + ss;
        while (now < valid.size() && valid[now].time <= curTime)
        {
            if (valid[now].state == in)numCar++;
            else numCar--;
            now++;
        }
        printf("%d\n", numCar);
    }
    /*int curNum = 0;//记录当前停的车
    int curIndex = 0;//记录遍历到的valid下标
    while (K--)
    {
        int hh, mm, ss;
        scanf("%d:%d:%d", &hh, &mm, &ss);
        int curTime = hh * 3600 + mm * 60 + ss;//当前时间
        //遍历记录

        for (int i = curIndex; i < valid.size(); i++)
        {
            //如果当前询问时间早于或等于这辆车的时间，那么就要输出询问
            if (curTime < valid[i].time)
            {
                curIndex = i;//下标不变
                printf("%d\n", curNum);
                break;//输出完当前询问的结果后跳出这个循环，去执行下个询问
            }
            else//如果i车还没到点，统计当前停的车的数量
            {
                if (valid[i].state == in) curNum++;
                else curNum--;//车要出去
            }
        }
    }*/
    //输出停车时间最长的车
    for (auto c : totalTime)
    {
        if (c.second == maxTime)//打印最久车
        {
            printf("%s ", c.first.c_str());
        }
    }
    printf("%02d:%02d:%02d", maxTime / 3600, maxTime / 60 % 60, maxTime % 60);
}
```

    这道题问题太多了，甚至现在虽然拿到满分了，但依旧有问题。先说说注意事项吧。

* 关于string的比对，如何判断两个string相等，其实用`==`就行，但我的程序在开始时，并不能正常判断，这是因为我为了能够使用`scanf`来接收string，把string给`resize()`了一下，这一下久导致了无法正确判断，因为resize后，其余位用`/0`补完了，所以在进行相等判断时，`state == “in”`，此时state为"in/0/0"，而in其实是"in/0"这就导致判断两个字符串不相等了，所以我修复的办法就是把"in"存到string里，也给`resize`一下，out的方法一样

* 其他还有很多小问题，比如漏写break（尽管现在的代码已经没break了），导致统计车数出错

* 还有就是打印问题，用printf打印不出东西来，后来我把前面所有的cout都改成printf就好了

* 最后一个问题，也是我到现在也不太清楚的问题：就是当前车辆数目计算的方法，我自己写的方法和算法笔记的方法含义一样，只不过我用的for循环，他用的while，但我的代码，测试点1、2就是过不去，他的代码能过去，这一点挺匪夷所思的，我的代码写在注释里了，但我不懂为啥不行。不知道未来能否解决这个问题。只能说以后如果遇到类似的问题，还是得精准控制想要使用的变量，尽量别交给for循环去++，当然，不出错的话就直接for循环没关系。



## A1092 [To Buy or Not to Buy](https://pintia.cn/problem-sets/994805342720868352/problems/994805374509498368)

```cpp
#include<iostream>
#include<string>
#include<unordered_map>
using namespace std;

int main()
{
	string shop, Eva;//商店的珠子和eva想要的珠子
	cin >> shop >> Eva;
	bool answer = true;
	//将商店的珠子存入哈希表
	unordered_map<char, int> mapShop;
	for (int i = 0; i < shop.size(); i++)
	{
		mapShop[shop[i]]++;
	}
	//去查是否能买够
	int lesscnt = 0;//缺的珠子数
	for (int i = 0; i < Eva.size(); i++)
	{
		if (!mapShop.count(Eva[i]) || mapShop[Eva[i]] <= 0)//如果想要的珠子没有
		{
			answer = false;
			lesscnt++;
		}
		else
		{
			mapShop[Eva[i]]--;
		}
	}
	if (answer)
	{
		cout << "Yes" << ' ' << shop.size() - Eva.size();
	}
	else
	{
		cout << "No" << ' ' << lesscnt;
	}
}
```

    简单的哈希表，第一次提交竟然没全对，忘了减商店的珠子了 ......
