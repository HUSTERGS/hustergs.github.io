---
title: cpp concurrency in action ch3
tags:
  - c++
  - concurrency
abbrlink: a1477cf6
date: 2021-11-24 16:36:48
---


> In concurrency, a race condition is anything where the outcome depends on the relative ordering of execution of operations on two or more threads;

因为race condition一般来说都是**时间敏感**(*timing-sensitive*)的，所以如果直接debug的话经常就无法复现

#### 避免race condition

主要有三种方法避免race condition

1. 使用锁之类的机制

2. 更改数据结构以实现无锁并发

3. 将对数据结构的更新作为一个**事务**(*transaction*)

   > 好像有相关的术语*software transactional memory (STM)*，可以了解一下？

<!-- more -->

即使使用了锁等机制，也可能由于**接口**(*interface*)本身的设计问题而导致race condition

1. 接口中返回了对数据的引用或者指针
2. 在接口中执行了用户自定义的函数，而该函数可以对被保护的数据进行操作
3. 接口本身的设计问题

主要对`3`进行进一步的说明

比如一个stack

```c++
stack<int> s; 
if(!s.empty()) {
	int const value=s.top(); 
  s.pop(); 
  do_something(value);
}
```

就会出问题，可能两个线程都认为没有空，但是实际上只有一个元素，结果pop了两次

#### 在C++中使用mutex

> 感觉所谓的RAII好像就是在对象的析构函数里面干一些事情，比如释放资源之类的，保证在异常发生的时候资源也可以被释放

1. `std::lock_guard`

   构建的过程就会上锁

   ```c++
   #include <list>
   #include <mutex>
   #include <algorithm>
   std::list<int> some_list;
   std::mutex some_mutex;
   void add_to_list(int new_value)
   {
     std::lock_guard<std::mutex> guard(some_mutex);
     some_list.push_back(new_value); 
   }
   bool list_contains(int value_to_find)
   {
     std::lock_guard<std::mutex> guard(some_mutex);
     return std::find(some_list.begin(),some_list.end(),value_to_find) != some_list.end();
   }
   ```

   C++17有一个新的特性称之为*class template argument deduction*，可以写成

   `std::lock_guard guard(some_mutex);`

   一般来说会将锁以及需要保护的数据结构整合在一起。（可能需要将锁定义为mutable，使得对于const的对象也可以上锁）

   > 可以使用`std::adopt_lock`作为第二个参数传入`lock_guard`，来接受一个已经锁上的锁

2. `std::unique_lock`

   不管是`std::lock_guard`还是`std::scoped_lock`，默认模式基本都是，构建的时候上锁，析构的时候解锁，但是在中间不能在对锁进行操作。但是`std::unique_lock`就提供了更多的灵活性，不仅与`std::lock_guard`一样可以接受`std::adopt_lock`来接受一个已经上锁的锁，也可以接受`std::defer_lock`来取消在构建的时候上锁。同时在构建之后也可以`lock`/`unlock`

#### 解决死锁

死锁的一个重要原因是锁住多个锁，然后在多线程的时候就有可能会有问题

1. `std::lock`

   可以一次性锁住多个锁

   ```c++
   class some_big_object;
   void swap(some_big_object& lhs,some_big_object& rhs);
   class X
   {
   private:
     some_big_object some_detail;
     std::mutex m;
   public:
     X(some_big_object const& sd):some_detail(sd){}
     friend void swap(X& lhs, X& rhs)
     {
       if(&lhs==&rhs)
         return;
       std::lock(lhs.m,rhs.m);
       std::lock_guard<std::mutex> lock_a(lhs.m,std::adopt_lock); // 使用std::adopt_lock来接受一个已经上锁的锁
       std::lock_guard<std::mutex> lock_b(rhs.m,std::adopt_lock);
       swap(lhs.some_detail,rhs.some_detail);
     }
   };
   ```

2. `std::scoped_lock`

   类似于`std::lock_guard`，但是可以同时锁住多个锁

   ```c++
   void swap(X& lhs, X& rhs)
   {
     if(&lhs==&rhs)
       return;
   	std::scoped_lock guard(lhs.m,rhs.m);
     swap(lhs.some_detail,rhs.some_detail);
   }
   ```



**避免死锁的一些指导方针**

1. 尽量避免在已经获得了一个锁的情况下，去获取另一个锁
2. 避免在获得一个锁的时候，**执行用户自定义的函数**
3. 如果`1`无法避免，那么不同的线程尽量以固定的顺序获得锁



#### protecting shared data during initialization

比如有这样一个需求，

```c++
std::shared_ptr<some_resource> resource_ptr;
void foo()
{
  if(!resource_ptr)
  {
    resource_ptr.reset(new some_resource); 
  }
  resource_ptr->do_something();
}
```

在调用`foo`时需要保证`resource_ptr`已经被正确设置

第一种做法:

```c++
std::shared_ptr<some_resource> resource_ptr;
std::mutex resource_mutex;
void foo()
{
  std::unique_lock<std::mutex> lk(resource_mutex);
  if(!resource_ptr)
  {
    resource_ptr.reset(new some_resource); 
  }
	lk.unlock();
	resource_ptr->do_something();
}
```

直接在函数中加锁，这样会导致所有的线程都线性的执行，完全丧失了并行的优势。

另一种非常**infamous**的做法是*double-checked locking*

```c++
void undefined_behaviour_with_double_checked_locking()
{
	if(!resource_ptr)
  {
    std::lock_guard<std::mutex> lk(resource_mutex);
    if(!resource_ptr)
    {
      resource_ptr.reset(new some_resource); }
  }
  resource_ptr->do_something();
}
```

中间的另一次对`resource_ptr`的检查是为了防止在前一个对`resource_ptr`的判断和获取锁的中间，已经有别的线程进行了初始化。但是即使如此，依然会有问题（如果没有问题就不会说是**infamous**了）

对`resource_ptr`的读和写没有进行同步

> 如果`resource_ptr`本身是原子量的话就可以吗？

使用`std::once_flag`以及`std::call_once`

```c++
std::shared_ptr<some_resource> resource_ptr;
std::once_flag resource_flag;
void init_resource()
{
	resource_ptr.reset(new some_resource); 
}
void foo() {
  std::call_once(resource_flag,init_resource);
	resource_ptr->do_something();
}
```



#### 读写锁

> The C++17 Standard Library provides two such mutexes out of the box, `std:: shared_mutex` and `std::shared_timed_mutex`. C++14 only features `std::shared_ timed_mutex`, and C++11 didn’t provide either.

> The difference between `std::shared_mutex` and `std::shared_timed_mutex` is that `std::shared_timed_mutex` supports additional operations (as described in section 4.3), so `std::shared_mutex` might offer a performance benefit on some platforms, if you don’t need the additional operations.

对于写者，使用`std::lock_guard<std::shared_mutex>`，对于读者，使用`std::shared_lock<std::shared_mutex>`



本章还提到了一些其他的东西，但是讲的很细，建议直接看原文

1. 对于双向链表，如果每一个节点都有一个锁，那么需要以固定的顺序获取锁，比如说遍历的时候，需要依次获得当前节点以及之后节点的锁，别的操作，比如插入也会有类似的需求，被称为*hand-over-hand locking*

2. hierarchy lock

   每一个锁具有一个*等级*，在获得一个锁之后，无法获得比当前锁等级更高的锁

3. recursive_mutex

   可重入锁，同一个线程可以lock多次，同时解锁的时候也需要unlock多次

   > 一般不建议使用
