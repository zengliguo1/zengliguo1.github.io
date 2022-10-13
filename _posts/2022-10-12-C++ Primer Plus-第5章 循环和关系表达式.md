---
title: C++ Primer Plus-第5章 循环和关系表达式
date: 2022-10-12 17:48:00 +0800
categories: [C++, C++ Primer Plus]
tags: [学习笔记]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: true
mermaid: true

---

    今天是把上一章剩下的一点看完了。这一章内容其实不多，也不难，但晚上要去做摄影作业了，又得拖到明天才能看完了。

    感觉EOF这里多看了会，别的问题不大。

## 5.1 for循环

- 步骤：
  
  1.设置初始值
  
  2.执行测试，看看循环是否应当继续进行
  
  3.执行循环操作
  
  4.更新用于测试的值

- C++常用的方式是，在for（和其他控制语句if、while）和括号之间加上一个空格，而省略函数名和括号之间的空格。这样从视觉上强化了控制语句和函数调用之间的区别

- **表达式**到**语句**的转换只需要加一个分号，所有表达式都可以成为**语句**，但不一定有编程意义：`rodents + 6;`

- 副作用（side effect）：指的是在计算表达式时对某些东西（如存储在变量中的值）进行了修改

- 顺序点（sequence point）：是程序执行过程中的一个顺序点

- 在C++中，语句中的分号就是一个顺序点，意味着程序处理下一条语句之前，赋值运算符、递增运算符和递减运算符执行的所有修改都必须完成。另外，任何完整的表达式末尾都是一个顺序点

- 完整的表达式：不是另一个更大的表达式的子表达式

- C++11文档中不再使用“顺序点”，因为难以用于讨论多线程。所以使用了“顺序”

- 然而，虽然选择使用**前缀格式**还是**后缀格式**对程序的行为没有影响，但执行速度可能有细微的差别。对于内置类型和当代的编译器而言，这看似不是任么问题。然而，C++允许您**针对类定义这些运算符**，在这种情况下，用户这样定义前缀函数：将值加1，然后返回结果：但后缀版本首先复制一个副本，将其加1，然后将复制的副本返回。因此，对于类而言，一前缀版本的效率比后缀版本高

- 指针递增和递减遵循指针的算术规则

## 5.2 while循环

- 由于for循环和while循环几乎是等效的，因此究竞使用哪一个只是风格上的问题。它们之间存在三个差别：
  
  - 首先，在for循环中省略了测试条件时，将认为条件为tnue;
  
  - 其次，在for循环中，可使用初始化语句声明一个局部变量，但在while循环中不能这样做；
  
  - 最后，如果循环体中包括continue语句，情况将稍有不同，continue语句将在第6章讨论。

- 通常，程序员使用for循环来为循环计数，因为for循环格式允许将所有相关的信息—初始值、终止值和更新计数器的方法一放在同一个地方。在无法预先知道循环将执行的次数时，程序员常使用while循环

- **编写延时循环**
  
  - 使用ctime头文件，其中定义了一个符号常量`CLOCKS_PER_SEC`，每秒钟包含的系统时间单位数（我测试了下我的是1000），因此，将系统时间除以这个值，可以得到秒数（其中可以使用clock()函数返回程序开始执行后所用的系统时间）
  
  - ctime头文件将clock_t作为clock()返回类型的别名，因为不同系统的clock()函数返回的类型可能不同（比如long或unsigned long）(确实有道理，毕竟时间不会是负数，无符号更合适)

## 5.3 do while循环

- do while属于出口条件循环

- 一般情况下，入口条件循环好

## 5.4 基于范围的for循环(C++11)

- 其实就是for循环使用`：`

- for(auto x : array){}

## 5.5 循环和文本输入

- 文本输入，需要选取某个特殊字符——哨兵字符(sentinel character)，将其作为停止标记

- 使用`cin.get(char)`可以吸收空格，但单cin不行

- 如果输入来自文件，则可以使用一种更强大的技术——检测文件尾(EOF)，C++输入工具和操作系统协同工作，来检测文件尾并将这种信息告知程序

- 检测到EOF后，cin将两位(eofbit和failbit)都设置为1。可以通过成员函数eof()来查看eofbit是否被设置；如果检测到EOF,则cin.eof()将返回bool值true，否则返回false。同样，如果eofbit或failbit被设置为1,则fail()成员函数返回true，否则返回false。注意，eof()和fail()方法报告最近读取的结果；也就是说，它们在事后报告，而不是预先报告。因此应将cin.eof()或cin.fail()测试放在读取后，程序清单5.18中的设计体现了这一点。它使用的是fail(),而不是eof(),因为前者可用于更多的实现中。**`ctrl + z` 和`Enter`可以模拟文件的EOF**(不同环境会有所不同，我的windows是可以的)
  
  ```cpp
  #include<iostream>
  using namespace std;
  int main()
  {
      char ch;
      cin.get(ch);
      int count = 0;
      while (cin.fail() == false)
      {
          cout << ch;
          ++count;
          cin.get(ch);
      }
      cout << endl << count;
  }
  ```

- 方法cin.get(char)的返回值是一个cin对象。然而，istream类提供了一个可以将istream对象（如cin)转换为bool值的函数：当cin出现在需要bool值的地方（如在while循环的测试条件中）时，该转换函数将被调用。另外，如果最后一次读取成功了，则转换得到的bool值为true:否则为false。这意味着可以将上述while测试改写为这样：`while(cin)`。这比`!cin.fail()`或`!cin.eof()`更通用，因为它可以检测到其他失败原因，如磁盘故障。
  
  ```cpp
  #include<iostream>
  using namespace std;
  int main()
  {
      char ch;
      int count = 0;
      while (cin.get(ch))
      {
          cout << ch;
          ++count;
      }
      cout << endl << count;
  }
  ```

- 可以使用`int ch`,并用`cin.get()`代替`cin.get(char)`,用`cout.put()`代替`cout`,用`EOF`测试代替`cin.fail()`测试：
  
  ```cpp
  #include<iostream>
  using namespace std;
  int main()
  {
      int ch;
      int count = 0;
      ch = cin.get();
      while (ch != EOF)
      {
          cout.put(ch);//cout.put(char(ch)) 对一些实现需要强制类型转换
          ++count;
          ch = cin.get();
      }
      cout << endl << count;
  }
  ```
* [![xdVKCd.jpg](https://s1.ax1x.com/2022/10/13/xdVKCd.jpg)](https://imgse.com/i/xdVKCd)
- 使用`cin.get(ch)`更符合对象方式

## 5.6 嵌套循环和二维数组

- 在输出中使用制表符比使用空格可使数据排列更有规则。（跟本节内容好像没啥关系，不过这节确实没啥内容）
