---
title: PAT-First Contact & Cut Integer
date: 2022-07-21 14:52:00 +0800
categories: [算法刷题, PAT]
tags: [模拟, 数学]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1139 [First Contact](https://pintia.cn/problem-sets/994805342720868352/problems/994805344776077312)

```cpp
#include<iostream>
#include<set>
#include<utility>
#include<unordered_map>
#include<vector>
using namespace std;
struct personInfo
{
    int gender;//1为男，0为女
    string name;
    set<string> maleFri;//男性朋友
    set<string> femaleFri;//女性朋友
    //构造函数
    personInfo(int _gender, string _name) :
        gender(_gender), name(_name)
    {
    }
};
int main()
{
    //读入数据
    int N, M;//N个人，M行关系
    cin >> N >> M;
    //通过名字映射结构体地址
    unordered_map<string, personInfo*> name2Info;
    //读取关系
    while (M--)
    {
        string p1, p2;
        cin >> p1 >> p2;
        if (!name2Info.count(p1))//如果没p1这个人
        {

            name2Info[p1] = new personInfo(p1[0] == '-' ? 0 : 1, p1);
        }
        if (!name2Info.count(p2))//如果没p2这个人
        {

            name2Info[p2] = new personInfo(p2[0] == '-' ? 0 : 1, p2);
        }
        //将对方加入自己的朋友中
        if (p2[0] == '-')//p2是个妹子
        {
            (name2Info[p1]->femaleFri).insert(p2);
        }
        else//p2是个汉子
        {
            (name2Info[p1]->maleFri).insert(p2);
        }
        if (p1[0] == '-')//p1是个妹子
        {
            (name2Info[p2]->femaleFri).insert(p1);
        }
        else//p1是个汉子
        {
            (name2Info[p2]->maleFri).insert(p1);
        }
    }
    //处理询问
    int K;
    cin >> K;
    while (K--)
    {
        int cnt = 0;
        string p1, p2;
        cin >> p1 >> p2;
        //如果p1或者p2并不存在于数据库中
        if (!name2Info.count(p1) || !name2Info.count(p2))
        {
            //最后一次的不换行
            if (K == 0) cout << 0;
            else cout << 0 << endl;
            continue;
        }
        vector<pair<string, string> > edge;//关系桥数组
        edge.clear();//每次询问清空数组
        set<string>& p1Fri = name2Info[p1]->gender == 0 ? name2Info[p1]->femaleFri : name2Info[p1]->maleFri;
        set<string>& p2Fri = name2Info[p2]->gender == 0 ? name2Info[p2]->femaleFri : name2Info[p2]->maleFri;
        for (auto& _p1Fri : p1Fri)//name2Info[p1]->gender == 0 ? name2Info[p1]->femaleFri : name2Info[p1]->maleFri)
        {
            for (auto& _p2Fri : p2Fri)//(name2Info[p2]->gender == 0 ? name2Info[p2]->femaleFri : name2Info[p2]->maleFri))
            {
                //如果p1的朋友的朋友中有p2的这个朋友
                if (name2Info[_p1Fri]->maleFri.count(_p2Fri) || name2Info[_p1Fri]->femaleFri.count(_p2Fri))
                {
                    string _p1FriName = _p1Fri[0] == '-' ? _p1Fri.substr(1) : _p1Fri;
                    string _p2FriName = _p2Fri[0] == '-' ? _p2Fri.substr(1) : _p2Fri;
                    //p1朋友的朋友不能是p1，也不能是表白对象p2(这个其实不用判断，因为p2Fri是p2的朋友，不可能是p2自己)
                    //p1朋友不能是表白对象
                    if (_p2Fri != p1 && _p1Fri != p2)
                    {
                        edge.emplace_back(make_pair(_p1FriName, _p2FriName));
                        cnt++;
                    }

                }
            }
        }
        //打印所有的朋友桥
        //如果最后一个是0，那么不换行
        if (cnt == 0 && K == 0) cout << 0;
        else
        {
            cout << cnt << endl;
            for (int i = 0; i < cnt; i++)
            {
                //如果是最后一个询问，而且是最后一组数据的话，就不换行
                if (K == 0 && i == cnt - 1) cout << edge[i].first << ' ' << edge[i].second;
                else cout << edge[i].first << ' ' << edge[i].second << endl;
            }
        }

    }
}
```

    一开始运行是有报错的：Access violation reading location，这是因为在询问中的名字可能之前从来就没出现过，却依然去哈希表中去寻找了，那么自然就会说是访问违规阅读地点。

    这道题开始只拿了24分，最后三个测试点没过去，还好有[昵称五个字](https://blog.csdn.net/a617976080 "昵称五个字")大佬的博客，他提到了注意事项：朋友的朋友不能是自己或者表白目标，朋友也不能是目标，这恰恰是最后三个测试点的特殊情况。还有就是我直接用了string来存名字，规避掉了他还提到的**0000和-0000是代表不同的性别**这个点。

## A1132 [Cut Integer](https://pintia.cn/problem-sets/994805342720868352/problems/994805347145859072)

```cpp
#include<iostream>
#include<string>
using namespace std;

int main()
{
    int N;//储存个数
    cin>>N;
    while(N--)
    {
        string num;
        cin>>num;
        int len = num.size();
        string numA = num.substr(0, len / 2);
        string numB = num.substr(len / 2);
        int Num = stoi(num);
        int NumA = stoi(numA);
        int NumB = stoi(numB);
        if(NumB == 0)
        {
            cout<<"No";
        }
        else
        {
            if(Num % (NumA * NumB) == 0)
            {
                cout<<"Yes";
            }
            else
            {
                cout<<"No";
            }
        }

        if(N != 0) cout<<endl;
    }
}
```

    这道题很简单，但一开始也出错了。

* 首先是没考虑到拆分成两个数字后，第二个可能是0的情况，不过提交后，会提示浮点错误，所以还好解决

* 第二个错误是，我开始担心A*B会超过int的范围，所以就先num/A后再取余B，这样的操作对于一个num本身就可以被（A\*B）整除的的来说无伤大雅，但可能会把不能被整除的给误判成Yes，因为int的取整机制。
