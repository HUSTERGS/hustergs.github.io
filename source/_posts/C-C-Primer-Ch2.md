---
title: C++ Primer Ch2
tags: C++
categories:
  - Programming Language
  - C++
abbrlink: 795955fa
date: 2020-10-03 10:09:38
---


1. 现代的C++程序最使用`nullptr`而不是`NULL`由于`NULL`实际上是一个预处理变量，使用`NULL`进行初始化与使用`0`进行初始化没有区别，因为`NULL`会在预处理阶段被朴素的文本替换为`0`。`nullptr`是C++11引入的新的字面量
2. 个人觉得C/C++的变量声明其实算是设计上的失误？
   ```c++
   int *p1, v; // 其中p1为指向int的指针，而v只是简单的int变量，这时候可以认为*是和p在一起的
   const int * p; // p指向的是一个不能被改变的int值，这时候const可以认为是和int在一起的？
   ```
   个人认为理解记忆起来还是有点绕
<!-- more -->
3. 默认状态下，`cons`t对象仅在单个文件中有效。
   当我们在不同文件中定义了同名的`const`变量的时候并不会发生冲突，而是理解为两个不同的`const`变量。如果想要在多个文件中使用统一个`const`变量，那么不管是声明还是定义都在其开始添加`extern`关键字

4. 常量引用是合法的`const &r = 0`
5. 善用`typedef`以及`auto`。同时需要注意使用`typedef`实际上是使新声明的变量与别名的**类型一致**，而不是简单的字符串替换
   
   ```c++
   typedef char *pstring;
   const pstring cstr = 0; // 指向char的常量指针
   const pstring *ps; // ps是一个指针，指向的对象是一个指向char的常量指针
   ```
   同时`auto`在作用于const变量时会丢弃顶层const而保持底层const
6. `decltype`
   ```c++
   decltype(f()) sum = x; // sum的类型是函数f返回类型，需要注意的是f函数并不会被计算
   ```
   与`auto`不同，`decltype`会同时保持顶层以及底层`const`。
   同时在`decltype`一节也可以真正看出引用和真正的变量本身还是有区别的，也就是引用真的是按照某种单独的类型进行处理的（本条所说的**引用*不是指针概念中的引用）
   ```c++
   int main() {
       int i = 42, *p = &i, &r = i;
       decltype(r) c;
       decltype(*p) m;
       return 0;
    }
   ```
   其中`r`是引用，所以c也是引用，c没有初始化，报错。同时需要注意的是解引用操作也会返回一个引用变量。同样返回引用变量的还有使用括号包起来的表达式。`decltype((variable))`返回的永远是一个引用，所以需要赋值
7. 可以在类定义时为成员提供初始值，未提供的将会以默认值初始化