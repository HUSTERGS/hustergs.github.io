---
title: cpp concurrency in action Ch1
tags:
  - c++
  - concurrency
abbrlink: 77045b3a
date: 2021-11-24 16:31:58
---


C++标准每三年制定一次
11, 14, 17



### 并发方式

主要有两种并发方式
1. task swichting
2. genuine concurrency

<!--more-->

前者通过在processor上快速切换任务来达到并发的"错觉"，而后者通过多个硬件core，来实现"真正"的并发。即使是在多核的处理器当中，也少不了task switching，因为同时运行的程序往往远远大于硬件所能支持的硬件并发数。

同时，后者相较于前者也会出现一些由于memory model所导致的新问题。task switch由于会涉及到操作系统层面的context switch从而不可避免的会存在额外的性能损失。所以如果一个执行单元的执行时间太短，甚至和task switch的时间对等，那么并发的性能可能甚至会低于非并发的性能

![image-20211122095434176](image-20211122095434176.png)

![image-20211122095512414](image-20211122095512414.png)



### 并发实现方式

具体来说，并发的实现方式主要有两种

1. 多进程
2. 多线程

前者的优势在于每一个进程相对来说较为独立，管理起来可能不容易出错，但是同时在不同进程之前进行交流的"代价"很高，同时操作系统启动一个进程的时间要远多于一个线程的时间。

> `Erlang`使用进程而不是线程作为并发的最小单位

后者在线程之间的交流之间有着巨大的优势----全局变量可以直接访问，同时指针或者引用可以在线程之间方便地传递。但同时，代码构建者也需要保证，如果数据被多个线程访问到，那么在任何时候这个数据都是consistent的。虽然线程相对于进程要轻量化很多，但也不是没有上限。由于每一个线程需要一个单独的栈空间（很好理解，至少每一个线程执行的函数的函数栈肯定是独立的），这对于32位的机器可能会出现问题，因为32位的机器每一个进程的地址空间只有4G。这时候一般使用线程池来一定程度地解决这个问题。

> 本书主要着重介绍后者



### 为什么使用并发

一般来说，有两种情况需要使用并发

1. 分离/解耦合
2. 性能需求

很好理解



### C++ 与并发

在c++11之前，并发的代码通常会使用一些platform-specific的代码，这就会导致代码的移植性变得很差。而C++11提供了统一的接口来抹平不同平台的差异。

> One particularly important design that’s common to many C++ class libraries, and that provides considerable benefit to the programmer, is the use of the Resource Acquisition Is Initialization (RAII) idiom with locks to ensure that mutexes are unlocked when the relevant scope is exited.

C++的并发库，很大程度上借鉴了Boost，同时Boost也随着C++新标准的发布逐步改变了自己的接口从而与新标准适配。

1. C++11

   memory model, atomic operation, threads

2. C++14

   a new mutex type for protecting shared data

3. C++17

   a full suite of parallel algorithms

虽然C++标准库对这些平台差异化的代码进行了抽象，但一般来说这种抽象是有代价的，意味着某些之前可能不需要执行的新代码段需要被执行，这被称为abstraction penalty，但一般来说C++标准库的性能还是非常好的，即使profiling显示性能瓶颈在标准库，比如mutex，那也可能是应用设计的问题，比如让太多的线程来同时竞争这一个锁，这时候与其想要提升mutex的性能，不如减少竞争mutex的线程数。

> 感觉有时候也不是那么好实现的



Although the C++ Thread Library provides reasonably comprehensive facilities for multithreading and concurrency, on any given platform there will be platform-specific facilities that go beyond what’s offered. In order to gain easy access to those facilities without giving up the benefits of using the Standard C++ Thread Library, the types in the C++ Thread Library may offer a `native_handle()` member function that allows the underlying implementation to be directly manipulated using a platform-specific API. By its nature, any operations performed using native_handle() are entirely platform dependent
