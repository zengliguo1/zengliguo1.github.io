---
title: PAT-Kuchiguse & Stucked Keyboard & getline()用法
date: 2022-06-21 17:25:00 +0800
categories: [算法刷题, PAT]
tags: [模拟]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## A1077. [Kuchiguse](https://pintia.cn/problem-sets/994805342720868352/problems/994805390896644096)

```cpp
#include<iostream>
#include<vector>
#include<string>
using namespace std;

int main()
{
    int N;//记录有几个句子
    int minLength = 257;//记录句子最短长度
    vector<char> stack;//当作栈使，存后缀
    vector<string> sentence;//记录每一条句子
    cin >> N;
    getchar();//吸收换行
    for (int i = 0; i < N; i++)
    {
        string tmp;
        getline(cin, tmp);
        //更新最短长度
        if (tmp.size() < minLength) minLength = tmp.size();
        sentence.emplace_back(tmp);//记录每个句子
    }
    bool isStop = false;
    //从每个字符的最后一个开始
    for (int i = 0; i < minLength; i++)
    {
        char last = sentence[0][sentence[0].size() - 1 - i];
        //查每个句子的对应的字符
        for (int j = 1; j < N; j++)
        {
            if (sentence[j][sentence[j].size() - 1 - i] != last)
            {
                isStop = true;
                break;
            }
        }
        if (isStop)//如果已经出现不同了
        {
            if (stack.empty())//说明从最后一个字符开始就不一样了
            {
                cout << "nai";
            }
            else//如果栈里有字符，那就反过来打印
            {
                while (!stack.empty())
                {
                    cout << stack.back();
                    stack.pop_back();
                }
            }
            break;
        }
        else//如果没有stop
        {
            stack.emplace_back(last);
        }
    }
    //如果遍历完了，说明最短的字符就是答案
    while (!stack.empty())
    {
        cout << stack.back();
        stack.pop_back();
    }

}
```

    这道题目就是从后往前遍历即可，稍微用到了栈，但我用vector当栈使了，中间有个编译错误，是因为我用vector\<string>类型的结构去接收char了。

    由于这道题再次使用到了getline()函数，那么在这里总结下getline()函数的用法吧

## getline()函数用法

有两种getline函数

### 1.头文件\<istream>中

**语法**

> istream &getline( char *buffer, streamsize num );
> istream &getline( char *buffer, streamsize num, char delim );

    getline()函数用于输入流，读取字符到*buffer*中，直到下列情况发生：

- *num* - 1个字符已经读入,
- 碰到一个换行标志，
- 碰到一个EOF，
- 或者，任意地读入，直到读到字符*delim*。*delim*字符不会被放入*buffer*中。

```cpp
#include <iostream>                    
using namespace std;

int main()
{
    char name[256];
    cout << "Please input your name: ";
    cin.getline(name, 256);
    cout << "The result is:   " << name << endl;

    return 0;

}
```

### 2.头文件\<string>中

**语法**

> istream& getline (istream&  is, string& str, char delim);  
> 
> istream& getline (istream&& is, string& str, char delim);  
> 
> istream& getline (istream&  is, string& str);
> 
> istream& getline (istream&& is, string& str);

* is    ：表示一个输入流，例如 cin。

* str   ：string类型的引用，用来存储输入流中的流信息。

* delim ：char类型的变量，所设置的截断字符；在不自定义设置的情况下，遇到’\n’，则终止输入。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main()
{
    string name;
    cout << "Please input your name: ";
    getline(cin, name);
    cout << "Welcome to here!" << name << endl;

    return 0;

}
```

## A1112.[Stucked Keyboard](https://pintia.cn/problem-sets/994805342720868352/problems/994805357933608960)

```cpp
#include<iostream>
#include<unordered_map>
#include<string>
#include<cmath>
using namespace std;
const int L = 256;
int main()
{
    int k;//记录重复的个数
    cin>>k;
    string sentence;
    cin>>sentence;
    bool isBroken[L] = {false};//用来判断这个键是否坏掉
    bool isShowed[L] = {false};//用来判断之前是否已经查过这个键
    bool isPrinted[L] = {false};//用来判断打印第一行时是否打印过
    int left = 0, right = 1;//双指针判断
    while(right < sentence.size())
    {
        if(isShowed[sentence[left]])//如果当前字符已经检查过
        {
            left++;
            right = left + 1;
            continue;
        }
        //当前字符没有检查过
        if(sentence[right] != sentence[left])//如果下一个字符和当前字符不一样
        {
            isShowed[sentence[left]] = true;//设置为检查过
            left++;
            right = left + 1;
            continue;
        }
        else//如果下一个字符和当前字符一样
        {
            //开始检查一样的个数有几个
            while(right < sentence.size() && sentence[right] == sentence[left])
            {
                right++;
            }
            if((right - left) % k == 0)//说明是坏键
            {
                isBroken[sentence[left]] = true;//设置为坏键
                isShowed[sentence[left]] = true;//设置为检查过
            }
            else //说明不是坏键
            {
                isShowed[sentence[left]] = true;//设置为检查过
            }
        }
        //此时，下一个字符和当前字符一样，并且已经处理完毕，可以继续去判断了
        left = right;
        right = left + 1;
    }
    //更新坏键，如果一开始判断为坏键，但后面又出现了正确的形式，那就应该再更新一次
    for(int i = 0; i < sentence.size(); i++)
    {
        if(isBroken[sentence[i]])//如果这个键是坏的
        {
            int j = i + 1;

            while(j < sentence.size() && sentence[j] == sentence[i]) j++;
            if((j - i) % k != 0) isBroken[sentence[i]] = false;//此时更新，坏键其实是好的
            i = j - 1;
        }
    }
   //先打印个坏键
    for(int i = 0; i < sentence.size(); i++)
    {
        //如果没被打印过，而且还是坏键
        if(!isPrinted[sentence[i]] && isBroken[sentence[i]])
        {
            cout<<sentence[i];
            isPrinted[sentence[i]] = true;
        }
    }
    cout<<endl;
    //打印正确的顺序
    for(int i = 0; i < sentence.size(); i++)
    {
        //如果没坏，照常打印
        if(!isBroken[sentence[i]]) cout << sentence[i];
        else
        {
            cout << sentence[i];
            i += k - 1;//跳过后面相同的字符（注意 - 1是因为下一次循环会+1）
        }
    }
}
```

    这道题，坑有点多......

* 首先是理解题目意思，只有连续出现k次或k的倍数次才算可能是坏键，如果连续出现了2*k - 1次，就不是坏键

* 还有就是，输出的时候，是每k个输出一次坏键

* 尽管判断是坏键了，但后面有可能平反，如果一开始判断是坏键但出现了一段不是k的倍数次的序列，那么就得更新为不是坏键比如**3 aaabbbcccaabb**（这就是测试点1为什么过不去）

* 还有要注意的点是，再后面更新坏键的循环中，更新后，i = j是不对的，因为下一步到了for循环，i还会+1，这样就会跳过一个字符，产生错误
