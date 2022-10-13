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

## 6.7 读取数字的循环

## 6.8 简单文件输入/输出


