---
title: gopl-ch8
abbrlink: 8fd86e1
categories:
  - Programming Language
  - Go
---

## Chapter 
> Goroutines and Channels

### Goroutines
1. 第一个`gorountine`为`main`
2. 使用`go`关键字，例子如下
   ```go
   package main

    import (
        "fmt"
        "time"
    )

    func main() {
        go spinner(100 * time.Millisecond)
        const n = 45
        fibN := fib(n)
        fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
    }

    func spinner(delay time.Duration) {
        for {
            for _, r := range `-\|/` {
                fmt.Printf("\r%c", r)
                time.Sleep(delay)
            }
        }
    }
    func fib(x int) int {
        if x < 2 {
            return x
        }
        return fib(x-1) + fib(x-2)
    }
   ```

3. 除了退出之外没有其他办法使得一个`gorountine`停止另外一个，当然，依然可以通过`gorountine`之间的交流来*请求*对方停止
4. 我们常常使用函数字面量的方式，来启动一个`gorountine`，即`go 函数定义(传入参数)`，而这个过程又常常涉及到闭包，由于函数字面量可能在函数体中引用外部的变量，如`channel`
### Example: Concurrent Clock Server
### Example: Concurrent Echo Server

### Channels
1. 创建一个`channel`
   ```go
   ch := make(chan int) // 创建了一个名为ch的channel，其中传递的数据类型为int
   ```

2. `channel`的基本操作
   - Send: 
   `ch <- x` ，将x的值传入`ch`
   - Receive
   `x = <-ch`, 从`ch`接收值并赋给变量`x`,也可以不设置接收对象，应该等同于`_ = <-ch`，即抛弃接收的值
   > 作为函数返回值的时候不需要声明临时变量，`return <- ch`即可
   - Close
   `close(ch)`将会关闭一个channel，对一个已经关闭的channel进行
     - `Send`操作将会产生`panic`
     - `Receive`操作将会获取可能已经传入的值，如果没有则获得的为对应类型的零值，(比如发送方在send之后立即关闭，此时可能接收方还没有收到，但channel已经被关闭，那么接收方依然可以对这个channel执行receive操作，直到没有值可以取出，可以理解为邮箱？)。在`Receive`操作中可以选择接收两个值，`x, ok = <-ch`其中第二个值 `ok`就可以用来表征是否已经取出了所有的值。当然由于这种需求非常普遍，所以go提供了`range`操作符，可以直接用来遍历通过channel传递的数据，在结束后自动跳出

channel又分为`buffered`/`unbuffered`两种，以下分别介绍这两种channel
#### Unbuffered Channels
1. 简单来说就是同步的，针对某一次发送，接收方没有确认接收的话，发送方将会被阻塞，对接收方也是一样，当尝试接收的时候，接收反进程将会一直等待到收到数据为止。

2. 其中go**确保**接收方的接收操作早于发送方被唤醒

3. 一般来说我们使用channel是为了传递信息，但`unbuffered channel`自身的同步特性使得我们可以以一种类似于同步锁的方式来使用channel，此时传递的数据其实不包含任何信息，数据类型常常选择一些简单的类型，如`int`

#### Buffered Channels
1. 基本形式
   ```go
   ch = make(chan int) // unbuffered channel
   ch = make(chan int, 0) // unbufferd channel
   ch = make(chan int, 3) // bufferd channel with capacity 3
   ```

    简单来说就是当缓冲区为空的时候，再次尝试接收，接收者会被阻塞，当缓冲区满了，再次尝试发送，发送者会被阻塞，其实很像**信号灯**。
2. 对buffered channel 可以使用`len`函数，返回缓冲区中元素的个数，同理可以使用`cap`获得缓冲区的大小



#### Unidirectional Channel Types
单项的channel。当函数接收channel作为参数的时候，往往同一个channel参数只会使用接收/发送操作，也即在该函数中，channel是**单向**的。
用法如下:
```go
func squarer(out chan<- int, in <-chan int)
```
其中out只能用于输出（向channel输入），in只能作为输入（从channel中获得）

#### Looping in Parallel
1. 
