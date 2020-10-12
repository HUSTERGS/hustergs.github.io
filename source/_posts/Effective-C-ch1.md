---
title: Effective C++ ch1
tags: C++
categories:
  - Programming Language
  - C++
abbrlink: 7194a4e1
date: 2020-10-09 09:35:47
---

> C++ primer 看到了智能指针什么的，并且之后的几章看的有点粗略，就没有做多少笔记，之后抽时间看，直接开始看effective c++
> 
由于书中的每一个`Item`涉及到的内容还是很多的，所以在每一个Item当中又会分出几个小点进行记录

1. 使用`const`，`enum`以及`inline`而不是`#define`
   **本节也可以被总结为，使用编译器而不是预处理器**
   - `const`
     `const`与`#define`最容易产生交叉的点在于对常量的定义，当我们想要定义一个常量的时候，我们可能使用以下两种方式
     ```c++
     #define ASPECT_RATIO 1.653

     const double AspectRatio = 1.653;
     ```
     由于在编译的过程中，所有的`ASPECT_RATIO`将会被完全替换为`1.653`。如果我们使用的是一个基于符号表的调试器，那么我们可能根本不知道1.653从何而来，尤其是在使用他人编写的头文件中定义了该宏的时候。但是使用`const`单独定义一个常量将不会有这个问题。而且也有某种可能上，使用`const`所生成的代码要比使用`#define`得到的代码更少，因为预处理器的无脑替换，有可能在最终生成的`.o`文件中有很多重复的`1.653`，而使用`const double`按理来说不会超出一个.
     <!-- more -->
     使用指针的时候需要注意顶层和底层都需要加上`const`
     ```c++
     const char * const authorName = "Scott Meyers"; // 本书作者
     // 或者更好的方式是下面这种
     const std::string authorName("Scott Meyers");
     ```
     > 如果我没有理解错的话，对于对象只需要使用一个const而无需担心声明的变量被指向另一个对象或者声明的变量自身发生改变。由于两者都被对象本身所具有的方法所控制，前者是运算符重载，后者是成员函数。对于一个const对象，其所能调用的成员函数是有限制的，必须是`const`函数至少使用string测试的时候是没问题的。

     在类中使用`const`定义的常量的时候，为了保证该成员不被重复定义，或者是为了节省空间？所以都会定义为`static const`

   - `enum`
     `enum`可以在类中实现针对于类的常量。与上述使用`static const`成员不同，`enum`要更接近与`define`，如enum的值是在编译的时候就确定的，也就是可以用来确定数组的长度等等，以及无法取得一个`enum`/`#define`的地址而可以取得一个`static const`的地址。
     ```c++
     class GamePlayer {
     private:
        enum {NumTurns = 5};
        int scores[NumTurns];
     }
     ```
     这种技巧被称之为`enum hack`
    
   - `inline`
     另一个使用`#define`的场景是通过`#define`来定义一个看起来是一个函数但是实际上并不需要付出函数调用所需的开销的宏
     ```c++
     #define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
     ```
     但是这种宏非常容易出问题，比如如下的使用情景

     ```c++
     inta=5,b=0;
     CALL_WITH_MAX(++a, b); // a is incremented twice
     CALL_WITH_MAX(++a, b+10); // a is incremented once
     ```
     `a`自增的次数竟然与其所比较的对象有关系
     然而使用`inline`以及模板可以让我们在获得同样运行开销之外还不需要担心上述的问题，同时还可以将其定义为类的私有成员函数或者静态函数，这是宏所不具备的能力
     ```c++
     template<typename T> 
     inline void callWithMax(const T& a, const T& b) {
        f(a>b?a:b);
     }
     ```
2. 尽量使用`const`
   - 当声明一个`const`的对象的指针的时候，具体的类型与`const`的前后顺序并不会有影响，也即没有区别。但是两种情况都有可能出现，所以还是需要知道
     ```c++
     void f1(const Widget *pw);
     void f2(Widget const * pw);
     ```
     > 两者都表示`pw`所指向的对象是不可修改的，个人倾向于使用第一种

   - STL中的迭代器是根据pointer建立的(STL iterators are modeled on pointers)，所以迭代器与`T *`类型的指针很像，如果将一个迭代器声明为`const`就像是声明了一个`T* const`类型的指针(此处不能将其看作是简单的文本替换，并不是`const T * `)，其自身不能指向别的对象，但是它所指向的对象可以发生改变，如果需要使指向的对象不能发生改变需要使用`const_iterator`
   - 函数返回一个`const`在大多数情况下是不合适的，但是有些情况下还是需要使用，如
     ```c++
     class Rational{...};
     const Rational operator*(const Rational& lhs, const Rational& rhs);
     ```
     可能第一次看的时候无法理解为何乘法返回的是一个`const`对象，原因在于可能发生如下情况
     ```c++
     Rational a,b,c;
     ...
     if (a * b = c) ...
     ```
     也就是可能发生在条件判断中打字错误，将`==`打成`=`的情况，如果将函数写成`const`就可以避免这种情况
     > 个人认为是在某些函数的返回值从功能上可以判定为一定为一个右值的时候就写成`const`?
   - 使用`const`成员函数
     有两个原因
     1. 标识那些函数可以在`const`上调用，有利于理解代码
     2. 提升C++程序的一个重要途径就是传递`const`引用(pass objects by reference-to-const)

     针对const成员函数有两种理解，第一种是被称之为物理不变（physical constness）const函数不应该改变任何非静态成员，这也是编译器所实现的方式，但是有些情况却并不能被编译器检测到
     ```c++
     class CTextBlock { 
     public:
        ...
        char& operator[](std::size_t position) const { return pText[position]; }
     private:
        char *pText; 
     };

     const CTextBlock cctb("Hello");
     char *pc = &cctb[0];

     *pc = 'J';
     ```
     其中的`[]`重载函数并没有对对象进行任何的修改，但是却返回了对象成员的一个引用（此处不是值的拷贝而是引用，后面的小节应该会详细介绍，并且上述代码在mmacOS Catalina下会出现bus error）。
     所以就有了第二种理解，被称之为逻辑不变（logical constness），其认为const成员函数也可以更改成员，但是不能让调用的客户程序发现这一点，这也是作者推荐我们理解。然而这样也引出了一个新的问题，虽然我们主观上像用第二种理解，但编译器确是按照第一种理解来的，我们并不能更改const对象的成员，所以这里就引出了`mutable`关键字。使用`mutable`关键字修饰的成员可以在const函数中进行修改。
   - 如何避免代码重复
     根据上述的说明，我们往往需要为一个类的某一个功能同时制定const类型以及非const类型的成员函数，而这两者的代码段可能完全相同。这里我们可以通过**强制类型转换**来解决这个问题。虽然说强制类型转换也不是什么好鸟，但是这里我们为了解决代码重复还是破例使用它。
     ```c++
     class TextBlock { 
     public:
        ...
        const char& operator[](std::size_t position) const {
            ... // do bounds checking
            ... // log access data
            ... // verify data integrity return text[position];
        }
        char& operator[](std::size_t position) {
            ... // do bounds checking
            ... // log access data
            ... // verify data integrity return text[position];
        }
        private: std::string text;
     };
     ```
     上述代码中的两个重载函数代码完全一致，可以通过以下代码去除代码重复
     ```c++
     class TextBlock { 
     public:
        ...
        const char& operator[](std::size_t position) const {
            ...
            ...
            ...
            return text[position]; 
        }
        char& operator[](std::size_t position) {
            return const_cast<char&>(
                static_cast<const TextBlock&>(*this) [position]
            );
        }
     };
     ```
     主要分析非const版本。在非const版本中，先通过`static_cast`将`TextBlock`类型的`*this`转化为`const TextBlock`，从而调用const版本的重载函数。得到的结果是一个`const char &`的数据，接着再通过`const_cast`将`const`去除，返回`char &`类型的结果，实现了代码复用。因为每次调用非const类型的函数就说明肯定有一个非const的对象，所以调用它的const函数不回出现什么问题，但是反过来就不一定了。如果我们在const函数中调用非const函数就有可能出问题。