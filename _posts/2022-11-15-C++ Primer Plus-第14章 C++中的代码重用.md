---
title: C++ Primer Plus-第14章 C++中的代码重用
date: 2022-11-22 15:32:00 +0800
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

    最近有点偷懒了，而且发现了一个挺严重的问题，就是之前看过的内容有点不记得了，所以单单是啃书，效果不是特别好，还是得结合实际的项目来，所以之后的笔记可能不会记得太详细了，以后的书籍阅读可能也就是以读和查阅为主，也许不会再系统详细的记录长篇大论了...

* 代码重用方法：
  
  * 公有继承
  
  * 本身是另一个类的对象的类成员：称为包含(containment)、组合(composition)、层次化(layering)
  
  * 私有或保护继承

## 14.1 包含对象成员的类

* 公有继承，类可以继承接口，可能还有实现，获得接口是is-a关系的组成部分

* 使用组合（其他类对象作为本类成员），类可以获得实现，但不能获得接口，不继承接口是has-a关系的组成部分

* 复习：使用`explicit`关键字可以关闭隐式转换
  
  ```cpp
  //防止
  //student s;
  //s = 5;
  引发的隐式转换
  explicit Student(int n):name{"Nully"}, scross(n){}
  ```

* C++和约束：C++包含让程序员能够限制程序结构的特性——使用explicit防止单参数构造函数的隐式转换，使用const限制方法修改数据，这样做的根本原因：在编译阶段出现错误优于在运行阶段出现错误

* 初始化顺序：当初始化列表包含多个项目时，项目的初始化顺序为它们被声明的顺序，而不是在初始化列表中的顺序

* 如果代码使用一个成员的值作为另一个成员的初始化表达式的一部分时，初始化顺序就非常重要

## 14.2 私有继承

* 使用私有继承，基类的公有成员和保护成员都将称为派生类的私有成员——这意味着基类方法将不会称为派生对象公有接口的一部分，但可以在派生类的成员函数中使用它们

* **包含**将对象作为一个命名的成员对象添加到类中，而私有继承将对象作为一个未被命名的继承对象添加到类中。我们用术语**子对象**(subobject)来表示*通过继承*或*包含添加的对象*

* 使用多个基类的继承被称为多重继承(multiple inheritance, MI)

* 对于继承类，初始化将使用类名而不是成员名来标识构造函数（`ArrayDb`是`std::valarray<double>`的别名
  
  ```cpp
  Student(const char * str, const double * pd, int n)
  :std::string(str), ArrayDb(pd, n){}
  ```

* 使用包含时将使用对象名来调用方法，而使用私有继承时将使用类名和作用域解析运算符来调用方法
  
  ```cpp
  double Student::Average() const
  {
      if(ArrayDb::size() > 0)
          return ArrayDb::sum()/ArrayDb::size();
      else
          return 0;
  }
  ```

* 访问私有继承基类的对象：使用强制类型转换，由于Student是从string继承来的，那么强制转换为string结果就是继承而来的string对象，接着使用this指针即可
  
  ```cpp
  const string & Student::Name() const
  {
      return (const string &) *this;
  }
  ```

* 访问基类的友元函数：同样是使用强制类型转换，使用派生类友元来访问基类友元
  
  ```cpp
  ostream & operator<<(ostream & os, const Student & stu)
  {
      os<<(const String &)stu;
  }
  ```

* 那么使用包含还是私有继承呢？大部分选择包含
  
  * 包含易于理解
  
  * 继承更抽象，容易出现问题，比如包含同名方法的独立的基类或共享祖先的独立基类
  
  * 包含能包括多个同类子对象，继承只能有一个

* 使用私有继承的场景：
  
  * 私有继承可以访问基类的保护成员，但包含就不行
  
  * 需要重新定义虚函数时

* **保护继承**：基类的公有成员和保护乘员都将称为派生类的保护成员

* 和私有继承的区别在于第三代派生类，私有继承第三代派生类不能使用基类的接口，因为二代全给私有了，但保护继承的第三代类依然可以使用基类的公有、保护接口，因为第二代只是变保护

* 使用保护和私有派生时，重新定义访问权限可以使得基类方法可以在派生类外使用
  
  * 在派生类建一个公有方法直接调基类的方法（保护、公有的）
  
  * 使用公有部分的using声明指出派生类可以使用特定的基类成员
    
    ```cpp
    class Student : private std::valarray<double>
    {
        public:
            using std::valarray<double>::min;
    }
    ```

## 14.3 多重继承

* 私有MI（多重继承）和保护MI可以表示has-a关系

* MI带来的两个重要的问题：
  
  * 从两个不同的基类继承同名方法
  
  * 从两个或多个相关基类那里继承同一个类的多个实例

* **从两个或多个相关基类那里继承同一个类的多个实例**：使得基类指针指向派生类对象中基类对象地址时出现二义性

* 使用虚基类(virtual base class)来解决上面这个问题
  
  * 虚基类使得从多个类（它们的基类相同）派生出的对象只继承一个基类对象
  
  * 在继承时使用关键字virtual
    
    ```cpp
    class Singer:virtual public Worker{}
    class Waiter:public virtual Worker{}
    ```
  
  * 如此一来，继承于singer和waiter的新派生类就只包含worker对象的一个副本了

* 为什么用“虚”
  
  * 虚函数和虚基类无明显关系，只是反对引入新关键字，所以这有点像关键字重载

* 为什么不抛弃将基类声明为虚的方式，而使虚行为称为MI的准则呢
  
  * 在一些情况下，需要基类的多个拷贝
  
  * 虚基类要求完成额外的计算
  
  * 虚基类操作麻烦

* 使用虚基类时，需要对类构造函数采用新方法，为了避免在构造函数的参数列表自动传递信息（即调用基类构造函数）时，通过多条途径传递的冲突
  
  * 如果不希望默认构造函数（当有多条路径时会选择默认）来构造虚基类对象，需要显式地调用所需基类构造函数：
    
    ```cpp
    SingerWaiter(const Worker & wk, int p = 0, int v = Singer::other)
    : Worker(wk),waiter(wk,p),singer(wk,v){}
    ```

* **调用哪个方法**
  
  * 在多继承中，如果每个直接祖先都有一个同名函数，如show()，就会出现二义性
  
  * 可以使用作用域解析运算符
  
  * 更好的方法是在派生类中重新定义函数，指出要使用哪个版本的函数
    
    ```cpp
    void singingWaiter::show()
    {
        singer::show();
    }
    ```
  
  * 在多次这样调用会出现问题，要么模块化（拆的细一点），要么设置为protected，详情见p559

* **当虚基类和非虚基类混合使用**
  
  * 该类将包含一个表示所有虚途径的基类子对象和分别表示各条非虚途径的多个基类子对象

* 如果某个名称优先于(dominates)其他所有名称，则使用它时，即便不适用限定符也不会导致二义性
  
  * 派生类中的名称优先于直接或间接祖先类中的相同名称

## 14.4 类模板

* 定义类模板：使用模板定义替换类（如Stack）声明，使用模板函数替换类的成员函数
  
  * 模板类使用`template <class Type>`或`template <typename Type>`开头
  
  * 以相同的方法让模板成员函数替换原有类方法
    
    ```cpp
    template <class Type>
    class stack
    {
        public:
            stack();
            bool isempty();
    };
    template <class Type>
    stack<type>::stack()
    {
        ...
    }
    template <class Type>
    bool stack<Type>::isempty()
    {
        ...
    }
    ```

* 模板的具体实现被称为实例化(instantiation)或具体化(specialization)

* 不能将模板成员函数放在独立的实现文件中
- 泛型标识符——例如Type——称为类型参数，它们类似于变量，只能是类型赋给它们

- 可以讲内置类型或类对象用作模板的类型，指针也可以

- 一种允许指定数组大小的简单数组模板
  
  ```cpp
  tmplate<class T, int n>
  class arrayTp
  {
  private:
      T ar[n];
  public:
      explicit arratTp(const T & v);
  };
  template<class T, int n>
  arrayTp<T,n>::arayTp(const T & v)
  {
      ...
  }
  ```

- 可以递归使用模板：`arrayTp<arrayTp<int, 5>, 10> twodee`

- 可以使用多个类型参数：`template <class T1, class T2>`

- 模板具体化：有隐式实例显式实例化和显式具体化，统称为具体化(specialization)，详情见p582

- 成员模板：模板可用作结构、类或模板类的成员

- 模板类的友元，详情见p588
  
  - 非模板友元
  - 约束（bound）模板友元，即友元的类型取决于类被实例化时的类型
  - 非约束（unbound）模板友元，即友元的所有具体化都是类的每一个具体化友元

- 模板别名（C++11）
  
  ```cpp
  typedef std::array<double, 12> arrd;
  arrd gallons;
  ```
  
  - C++11允许使用模板提供一系列别名
    
    ```cpp
    template<typename T>
      using arrtype = std::array<T, 12>;
    ```
  
  - C++允许讲语法using＝用于非模板，此时与typedef等价
    
    ```cpp
    typedef const char * pc1;
    using pc2 = const char *;
    ```