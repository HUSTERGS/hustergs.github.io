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

### Example: Concurrent Clock Server
