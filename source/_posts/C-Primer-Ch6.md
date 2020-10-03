---
title: C++ Primer Ch6
tags: C++
categories:
  - Programming Language
  - C++
abbrlink: 7e3491e3
date: 2020-10-03 10:09:55
---
> 跳过了4,5两章，着实没有什么好记录的
1. 函数传递默认为值传递，也就是传入的变量会被复制一遍，使用引用为引用调用，原变量会被直接传入，从而避免拷贝，在传入大型类或者容器的时候非常有必要。C中一般使用指针，但是C++中建议将指针都改为引用，但是需要注意的是如果传入的变量在函数调用过程中不会被改变，那么最好是传递一个常量引用
<!-- more -->
2. 如果函数返回的是一个引用，将不会发生变量拷贝
   ```c++
   const string &shorterString(const string &s1, const string &s2) {
       return s1.size() <= s2.size() ? s1 : s2;
   }
   ```

3. 使用`NDEBUG`预处理变量来调整程序行为，如`assert`有时可以在编译的时候就很方便的定义，而不需要手动修改源码
   `$ CC -D NDEBUG main.C`

4. 使用`constexpr`标识一个函数返回的值是一个常量。同样在接受该函数的返回值所用的变量在声明的时候也需要使用`constexpr`
5. 内联函数