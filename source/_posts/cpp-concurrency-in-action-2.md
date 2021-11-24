---
title: cpp concurrency in action ch2
tags:
  - c++
  - concurrency
abbrlink: d6404c60
date: 2021-11-24 16:33:45
---


可以通过定义`operator()`来使一个对象变成可调用对象，从而可以作为thread的构造函数的参数来使用
```c++
class background_task
{
public:
    void operator()() const
    {
        do_something();
        do_something_else();
    }
};
background_task f;
std::thread my_thread(f);
```
<!-- more -->
在使用上述方式创建thread的时候，可能会遇到一个小问题，因为传入thread的`background_task`对象，可以是具体的具名对象，如例子中的`f`，但也可以是临时对象，比如`background_task()`，如果是后者就会跟c++的函数声明产生冲突

```c++
std::thread my_thread(background_task()); // 声明了一个函数
// 有以下两种解决方法
std::thread my_thread((background_task())); 
std::thread my_thread{background_task()};
```

如果说在主线程结束的时候，新建的线程还没有结束，那么新的线程对象的析构函数将会被调用，从而调用`std::terminate()`使新线程结束。

> 如果thread在heap上又会怎么样呢

所以就需要保证，在主线程结束（可能正常/异常结束）的时候，所有构件的对象都被`join()`或者`detach()`否则的话就会发生上面的事情。

有两种方式可以解决这个问题

1. 手动`try_catch`
2. 使用另一个类来包装thread，在其析构函数中调用thread的join

> 因为当一个函数结束的时候，局部变量是按照构建的顺序的反方向进行析构的



另一个容易出错的地方和上述的情况类似，如果在主线程结束的时候，新建的线程依然在访问主线程中的临时变量，就会导致问题。

```c++
struct func {
  int& i;
  func(int& i_):i(i_){}
  void operator()()
  {
    for(unsigned j=0;j<1000000;++j)
    {
      do_something(i);
    }
  }
};

void oops() {
  int some_local_state=0;
  func my_func(some_local_state); 
  std::thread my_thread(my_func); 
  my_thread.detach();
}
```



### 线程的参数传递机制

在C++的线程中，参数的传递分为两个步骤

1. 将参数**复制**到新thread对应的内存空间中
2. 通过复制之后的参数调用函数

当使用move的时候，以上的复制都会变成move

这样的话就会导致一些问题

**隐式转换**

```c++
void f(int i,std::string const& s);
void oops(int some_param)
{
	char buffer[1024];
	sprintf(buffer, "%i",some_param);
	std::thread t(f,3,buffer);
	t.detach();
}
```

线程本身并不会在意相关联的函数的参数类型，比如例子中的`f`，线程只负责将传入的参数传给函数，那么在隐式转换的时候（也就是函数被调用的时候），`buffer`可能已经被销毁了，这时候函数`f`通过隐式转换将`buffer`转换为`string`就会出错，所以就需要在创建线程的时候显式的进行转换

```c++
std::thread t(f,3,std::string(buffer));
```



---

可以通过`std::thread::hardware_ concurrency()`来获得硬件可以支持的并行数

> 在64核128线程的服务器上输出为64

`std::this_thread::get_id()`可以返回线程的唯一标识，标准库仅仅保证每一个线程的标识是唯一的，并且`std::thread::id`类型是可以进行比较的
