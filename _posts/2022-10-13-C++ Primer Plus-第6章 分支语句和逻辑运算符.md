---
title: C++ Primer Plus-第6章 分支语句和逻辑运算符
date: 2022-10-13 17:25:00 +0800
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

这章重点是简单文件输入输出，看来后面还会有复杂文件输入输出啊.....

## 6.1 if语句

- 条件运算符和错误防范：很多程序员将更直观的表达式variable == value反转为value==variable，以此来捕获将相等运算符误写为赋值运算符的错误。
  
  > 一般来说，编写让编译器能够以发现错误的代码，比找出导致难以理解的错误的原因要容易的多

## 6.2 逻辑表达式

- 也就是与或非，c++不需要头文件就可以使用标识符来表示：
  
  [![xdZYsx.jpg](https://s1.ax1x.com/2022/10/13/xdZYsx.jpg)](https://imgse.com/i/xdZYsx)

## 6.3 字符函数库cctype

- C++从C语言继承了一个与字符相关的、非常方便的函数软件包，它可以简化诸如确定字符是否为大写字母、数字、标点符号等工作，这些函数的原型是在头文件`cctype`(老式的风格中为ctype.h)中定义的。
  
  [![xdZyQI.jpg](https://s1.ax1x.com/2022/10/13/xdZyQI.jpg)](https://imgse.com/i/xdZyQI)

## 6.4 ?运算符

- 其实就是三目运算符

## 6.5 switch语句

- 可以使用枚举量作为标签

- 然而，如果所有的选项都可以使用整数常量来标识，则可以使用switch语句或ifelse语句。由于switch语句是专门为这种情况设计的，因此，如果选项超过两个，则就代码长度和执行速度而言，switch语句的效率更高。
  
  *提示：如果既可以使用if else if语句，也可以使用switch语句，则当选项不少于3个时，应使用switch语句。*

## 6.6 break和continue语句

- continue导致该程序跳过循环体的剩余部分，但不会跳过循环的更新表达式，也就是说，在for循环中，continue先跳到更新表达式处，然后跳到测试表达式处

## 6.7 读取数字的循环

- 输入错误和EOF都会导致cin返回false

- 读取错误发生的5种情况：以int类型的n输入单词为例
  
  - n不变
  
  - 不匹配的输入将被留在输入队列中
  
  - cin对象种的一个错误标记被设置
  
  - 对cin方法的调用将返回false
  
  - 接下来的cin拒绝输入，必须使用`cin.clear()`重置cin后才可以输入，而且会首先接受刚才被留在输入队列种的输入

- 当程序发生输入错误时，应该采取三个步骤：
  
  - 重置cin以接受新的输入
  
  - 删除错误输入
  
  - 提示用户再输入
    
    ```cpp
        int a;
        while (!(cin >> a))
        {
            cin.clear();
            while (cin.get() != '\n')//或者' '
                continue;
            cout << "please enter a number";
        }
    ```

## 6.8 简单文件输入/输出

- **写入文件：**
  
  - 必须包含头文件fstream，头文件fstream定义了一个用手处理输出的ofstream类
  
  - 需要声明一个或多个ofstream变量（对象），并以自己喜欢的方式对其进行命名，条件是遵守常用的命名规则
  
  - 必须指明名称空间std;例如，为使用用元素ofstream,必须使用编译指令using或前缀std
  
  - 需要将ofstream对象与文件关联起来，为此，方法之一是使用`open()`方法
  
  - 使用完文件后，应使用方法`close()`将其关闭
  
  - 可结合使用ofstream对象和运算符<<来输出各种类型的数据
  
  ```cpp
  #include<iostream>
  #include<fstream>
  #include<string>
  using namespace std;
  int main()
  {
      ofstream fout;
      string s;
      double d;
      fout.open("a.txt");
      getline(cin, s);
      cin >> d;
      fout.precision(2);//设置有效位为2（如果是1，那么就写入1.0）
      fout.setf(ios_base::showpoint);//强制写入小数点，如果不加这一行1就是写入1了
      fout << s << endl << d;
      fout.close();
  }
  ```

- 程序运行前，如果文件不存在，那么open就会新建一个文件；如果存在，那么清空该文件，重新写入

- **读取文件：**
  
  - 必须包含头文件fstream，头文件定义了一个用于处理输入的ifstream类
  
  - 需要声明一个或多个ifstream变量（对象），并以自己喜欢的方式对其进行命名，条件是遵守常用的命名规则
  
  - 必须指明名称空间std;例如，为使用用元素ifstream,必须使用编译指令using或前缀std
  
  - 需要将ofstream对象与文件关联起来，为此，方法之一是使用`open()`方法
  
  - 使用完文件后，应使用方法`close()`将其关闭
  
  - 可结合使用ifstream对象和运算符>>来读取各种类型的数据
  
  - 可以使用ifstream对象和`get()`方法来读取一个字符，使用ifstream对象和`getline()`来读取一行字符
  
  - 可以结合使用ifstream和`eof()`、`fail()`等方法来判断输入是否成功
  
  - ifstream对象本身被用作测试条件时，如果最后一个读取操作成功，它将被转换为布尔值truc,否则被转换为fase
  
  ```cpp
    #include<iostream>
  #include<fstream>
  #include<string>
  #include<cstdlib>
  using namespace std;
  int main()
  {
      ifstream fin;
      fin.open("a.txt");
      if (!fin.is_open())
      {
          cout << "No file";
          exit(EXIT_FAILURE);
      }
      int a, cnt = 0;
      fin >> a;
      while (fin.good())
      {
          cnt++;
          cout << a<<endl;
          fin >> a;
      }
      cout << cnt << endl;
      if (fin.eof())
          cout << "End of File\n";
      else if (fin.fail())
          cout << "data mismatch\n";
      else
          cout << "unknwon erro\n";
      fin.close();
  }
  ```

- 检查文件是否被打开的首先方法是使用方法`is_open()`

- 函数`exit()`(终止程序)原型在头文件cstdlib种定义的，该头文件中还定义了一个用于同操作系统通信的参数值`EXIT_FAILURE`

- 注意，文件中，最后一个信息之后最好加个空格或者回车，不然读不进去，比如`1 2 3`，最后的3是打印不出来的，此时 `fin.good()`已经返回false了，所以要在3后面加个空格或者回车。但是，如果不用`fin.good()`，而是直接用`fin>>a`作为条件，文件最后就不需要额外加东西了：
  
  ```cpp
      int a, cnt = 0;
      while (fin>>a)
      {
          cnt++;
          cout << a<<endl;
      }
  ```
