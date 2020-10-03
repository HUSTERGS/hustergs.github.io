---
title: C++ Primer Ch7
tags: C++
categories:
  - Programming Language
  - C++
abbrlink: 933a175
date: 2020-10-03 10:10:00
---

1. 成员便变量出现的顺序在成员函数之后，成员函数依然可以使用该成员变量，由于编译器是按照先成员变量再成员函数来处理的
2. 成员函数默认为`inline`
3. `this`是一个常量指针，指向的对象不是常量，不能对`this`本身进行修改
4. `struct`与`class`的区别在于默认访问权限不同，`struct`的默认访问权限为`public`，而`class`的默认访问权限为`private`
<!-- more  -->
5. 通过友元(`friend`)来使类的成员函数之外的函数能够对类有访问权限，如单独的函数或者其他类的成员函数
6. 常量成员函数
   `std::string isbn() const {return this->bokkNo;}`
   在参数列表之后加上`const`关键字表示该函数为一个常量成员函数。在常量成员函数中，对应的`this`变量的类型可以理解为`const className const * this`。也即不仅仅`this`本身是不能修改的，而且`this`所指向的实例也是不可以更改的，这时候就会引出一个问题，如果通过调用实例的某个成员函数对实例进行修改会发生什么？实际上，在常量成员函数中，能够通过`this`调用的成员函数也只能是常量成员函数。
7. 紧跟着第6点。我们有时候的确需要在常量成员函数中修改某一个成员的值，那么可以将该成员定义为`mutable`，定义为`mutable`的成员永远不会是`const`，即使某一个实例本身是`const`的，依然可以修改该成员。`mutable size_t access_ctr;`
8. 构造函数初始值列表
   `Sales_data(const std::string &s): bookNo(s), units_sold(0), revenue(0) {}`
9.  在const函数中如果以*this的形式返回，那么返回的类型是一个常量引用
10. 建议：
    对于公共代码使用私有功能函数。在实践中，设计良好的C++代码常常包含大量的类似于`do_display`的小函数，通过调用这些小函数可以组成大的函数。有以下好处
    - 避免在多处使用同样的代码
    - 随着模块的发展，这些重复的代码有可能变得更加复杂，整合成函数对于后续的修改和拓展都比较方便
    - 可以很容易的在`do_display`函数中添加某些调试信息，只需要在某一处增加和删除这些调试信息会方便很多
    - 额外的调用不会增加任何开销，由于我们在一个类的内部定义了`do_display`这个函数，所以它隐式的声明为内联函数，这样的话，调用该函数就不会带来任何的额外开销


11. 不仅仅有友元函数，也有友元类，友元类的成员函数可以访问此类的所有成员，包括私有成员
12. 类型成员
    ```c++
    class Screen {
    public:
        typedef std::string::size_type pos;
        // 或者使用 using
        using pos = std::string:size_type;
    private:
        pos cursor = 0;
        pos height = 0, width = 0;
        std::string contents;
    }
    ```
    需要注意的是，与普通成员不同，类型成员必须先定义再使用
13. 成员初始化的顺序与其在类中声明的顺序相同，以下代码会发生错误
    ```c++
    class X {
        int i;
        int j;
    public:
        X(int val) : j(val), i(j) {}
    }
    ```
    虽然在构造函数的构造列表中先写了`j`再写了`i`，但是由于实际声明是`i`在前，而`j`在后，所以就导致`i`使用了未定义的值`j`。所以尽量保证构造函数初始值的顺序与其声明的顺序一致，并且尽可能的避免使用某些成员初始化其他的成员。
    
14. 建议：
    在很多类中，初始化和赋值的区别事关底层效率问题：前者直接初始化数据成员，后者则直接初始化再赋值。除了效率问题之外，一些数据成员必须被初始化，比如引用类型的数据成员。所以尽量养成使用构造函数初始值的习惯
15. 委托构造函数
    > C++11
    ```c++
    class Sales_data {
    public:
        Sales_data(std::string s, unsigned cnt, double price): bookNo(s), units_sold(cnt), revenue(cnt * price) {}
        Sales_data() : Sales_data("", 0, 0) {}
        Sales_data(std::string s) : Sales_data(s, 0, 0) {}
        Sales_data(std::istream &is) : Sales_data() {
            read(is, *this);
        }
    }
    ```
    可以将某一个构造函数的部分或者所有的职责委托给别的函数
16. 隐式的类类型转换
    与之前提到的原子类型的隐式类型转换一样（int转换为double等），类也可能发生隐式类型转换。如15.中定义了一个接收`string`类型的`Sales_data`构造函数，那么在需要`Sales_data`的地方，如函数传餐，变量赋值等地方。如果传入了`string`类型的值，该值将会被自动转化为`Sales_data`。如
    ```c++
    string null_book = '9-999-99999-9'
    item.combine(null_book) // 在这一步null_book被自动用于构建一个Sales_data对象并传入item.combine函数
    ```
    同时需要注意隐式转换只允许一步。比如，不能从字符串字面量转换为`string`，再从`string`转化为`Sales_data`。但是有时候我们并不想这样的隐式转换发生，可以通过在对应的构造函数之前加上`explicit`关键字来限制隐式转换的行为。由于只有接收单一参数的构造函数才可能导致隐式转换发生，所以在接收多个参数的构造函数之前加上`explicit`关键字并没有什么意义。
