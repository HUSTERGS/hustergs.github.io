---
title: Effective C++ ch2
tags: C++
categories:
  - Programming Language
  - C++
abbrlink: e89df55b
date: 2020-10-12 20:48:00
---


1. 禁止拷贝（或其他操作）
   一般来说，对于普通的功能，我们不想要支持只需要不实现即可，但是对于赋值等操作，如果我们不实现，那么编译器会帮我们自动生成。所以我们可以通过将拷贝函数设定为`private`的方式来避免调用，然而这样还是可能会出问题，因为友元函数/类的存在，所以还是有机会调用这个函数，所以我们可以通过只声明而不定义拷贝函数，使得调用者在链接的时候报错，从而避免拷贝。

   **C++11支持使用`delete`关键字来非常方便的实现上述功能**
   <!-- more -->
2. 由多态以及析构函数所引发的一些问题
   考虑如下代码
   ```c++
   class TimeKeeper { 
   public:
        TimeKeeper( ); 
        ~TimeKeeper( ); 
        ...
   };
   class AtomicClock: public TimeKeeper { ... }; 
   class WaterClock: public TimeKeeper { ... }; 
   class WristWatch: public TimeKeeper { ... };
   ```
   当我们编写了一个工厂函数（在函数内部使用动态内存创建一个对象，该对象由调用者负责销毁），返回某一个指向子类的指针的时候，使用基类`TimeKeeper`来接收
   ```c++
   TimeKeeper* getTimeKeeper();
   ```
   这是多态的典型用法之一？但是注意基类`TimeKeeper`有一个非虚析构函数(non-virtual destructor)。那么在用户尝试销毁的时候，可能只有基类的部分被销毁，而子类的部分并没有被销毁（不是很清楚为何原文说的是**可能**）
   想要解决这个问题也非常简单，将基类的析构函数写成虚函数即可
   > 一般来说，如果一个类没有虚函数，那么这个类就不应该作为基类

   另一方面，如果一个类并不是作为基类设计的，而将其析构函数写成了虚函数，也会造成不好的影响，考虑如下类
   ```c++
   class Point { 
    public:
        Point(int xCoord, int yCoord); 
        ~Point( );
    private: 
        int x, y;
    }
   ```
   如果`int`类型是`32`位，那么一个`Point`类刚好可以放进一个64位的寄存器。并且这样一个类可以很方便的传递给其他的语言，比如C或者Fortran。如果我们将析构函数定义为虚函数，那么情况就不一样了。为了使虚函数可以正确工作，每一个对象需要有一个**虚表指针**，用于指向**虚函数表**，从而在调用的时候确定到底使调用的哪一个函数。在一个64位的架构上，Point类型的大小将会变为128位，整整增加了100%。point类不仅不再能够装入寄存器，它也不再具有了与其他的语言沟通的能力，因为其他的语言并不能理解虚表指针。
   总的来说，不论是习惯性的将析构函数定义为虚函数还是定义为非虚函数都有可能产生问题。
   同时由于继承的存在，对于一些具有非虚析构函数的类，我们最好也不要尝试定义其子类，如`string`，以及标准库中的大多数容器。

   对于抽象类，由于抽象类必须有一个纯虚函数，而抽象类一般来说就是为了作为基类而被设计的，所以自然而然的，最好的方法就是将析构函数定义为纯虚函数。需要注意的需要提供对应的定义，因为子类在调用析构函数的时候会也会调用基类的析构函数
   ```c++
   class AWOV { // AWOV = “Abstract w/o Virtuals” public:
        virtual ~AWOV() = 0; // declare pure virtual destructor 
    };
   AWOV::~AWOV() {} // definition of pure virtual dtor
   ```

3. 一般来说我们尽量不要强行压制一个**异常**。但如果我们在析构函数中有可能发生异常的时候，如连接/关闭数据库的操作，最好还是通过`try/catch`来进行压制，因为一个类析构失败将会导致使用该类为成员的类的销毁也失败，以及装载该类的容器的销毁也会失败，等等诸如此类。当然更好的解决办法是在对应的类中加上一个指示性的成员，用来标识操作相关的操作是否成功
4. 不要在构造函数或者析构函数中调用虚函数。因为在派生类构建的时候，会先调用基类的构造函数，而此时派生类的成员都还没有初始化，所以显而易见的是，此时调用的虚函数将会是基类中的版本，而不是派生类中的版本。
   > 在Java中也不能在构造方法中调用抽象函数

5. 赋值重载函数应该始终返回`*this`的引用，虽然不返回引用也可以编译成功，但是由于所有的内置类都是这样实现的，所以我们最好也这样做
6. 在赋值重载函数中处理对自身赋值情况的处理
   考虑如下代码 
   ```c++
   class Bitmap{...};
   class Widget {
    ...
    private:
        Bitmap *pb;
   }

   Widget&
   Widget::operator=(const Widget& rhs) {
       delete pb;
       pb = new Bitmap(*rhs.pb);
       return *this;
   }
   ```
   当我们将一个`Widget`对象赋值给它自身的时候，成员`pb`将会先被delete，此时`rhs.pb`将会指向一个无意义的对象，新的`pb`也会随之出现问题。对于这种情况我们一般有两种解决方法
   - 在函数的开始进行相等性的检查
     ```c++
     Widget&
     Widget::operator=(const Widget& rhs) {
        if (this == &rhs) return *this;
        delete pb;
        pb = new Bitmap(*rhs.pb);
        return *this;
     }
     ```
     但是即使如此依然会有异常的问题，比如new操作出了问题，那么pb将指向一个被删除的对象。同时需要注意的是相等性检查并不是免费的，我们同样需要考虑它的开销，可以结合自身拷贝这种情况发生的概率来做综合的决策
   - 仔细调整函数体中敏感操作（如上述`delete`）的顺序
     ```c++
     Widget& Widget::operator=(const Widget& rhs)
     {
        Bitmap *pOrig = pb;
        pb = new Bitmap(*rhs.pb); 
        delete pOrig;
        return *this;
     }
     ```
     先保存之前的pb，如果new失败了pb也可以保持之前的状态
   
   > 自身拷贝并不一定以上文中所述的那么直白的方式展现出来，在数组下标赋值`a[i] = a[j]`甚至在不同的类之间也有可能出现类似的问题（基类和派生类）

