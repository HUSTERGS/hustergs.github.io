---
title: gopl-ch5
tags: Go
categories:
  - Programming Language
  - Go
abbrlink: 764cfa5c
date: 2020-04-08 16:06:16
---


## Chapter 5
> Functions

### Function Declarations
1. 基本形式
   ```Go
   func name(parameter-list) (result-list) {
     body
   } 
   ```
   相同类型的参数或者返回值可以进行合并
   ```Go
   // 两者等价
   func f(i, j, k int, s, t string)
   func f(i int, j int, k int, s string, t string)
   ```
   一个函数的签名包括**参数和返回值**的类型,顺序,个数
2. Go中不存在参数的默认值,也无法根据参数名来传递参数,所以必须严格按照参数的顺序以及类型进行传递
3. 如果遇到没有函数体的函数,说明这个函数是由非Go语言实现的,而这个只是一个声明
<!-- more -->
### Recursion
1. 在Go中使用递归不需要担心栈溢出!(多么好的消息).与其他语言不同,Go并没有使用固定长度的栈,典型的Go实现使用变长的栈,可以使用上GB的空间用于递归

### Multiple Return Values
### Errors
### Function Values


### Anonymous Functions
> Go版本的lambda函数
1. 省略掉函数名的函数即为匿名函数,可以作为参数传递给诸如map等函数
   ```Go
   strings.Map(func(r rune) rune {return r + 1}, "HAL-9000")
   ```
2. 作为闭包使用, 匿名函数可以访问到外部函数的局部变量
   ```Go
   func squares() func() int {
       var x int
       return func() int {
           x ++
           return x * x
       }
   }

   func main() {
       f := squares()
       fmt.Println(f()) // "1"
       fmt.Println(f()) // "4"
       fmt.Println(f()) // "9"
       fmt.Println(f()) // "16"
   }
   ```
   > 个人感觉就像是x被强制导出了
3. 由于惰性求值所带来的问题,考虑如下例子
   ```Go
   var rmdirc []func()
   for _, d := range tempDirs() {
       dir := d // 必要的
       os.MkdirAll(dir, 0755)
       rmdirs = append(rmdirs, func() {
           os.RemoveAll(dir)
       })
   }

   for _, rmdir := range rmdirs {
       rmdir()
   }
   ```
   其中第一个循环中的`dir := d`是必要的.原因如下. 在`for`循环当中,并不是每一个循环都创建一个新的变量,而是使用的同一个变量,只是每次赋给了不同的值.也就意味着这个变量的地址始终不变. 而通过匿名函数使用他的时候,同样是通过地址来标示,并且由于函数具有惰性求值的特性,直到最后的`for`循环之前,`dir`的值都不会确定下来,如果没有使用`dir`进行缓存的话,那么最后的结果就是不断的删除同一个文件夹

### Variadic Functions
在变量类型之前加入`...`表示接收变长个参数,作为一个切片传入
```Go
func sum(vals ...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}
```
### Deferred Function Calls
1. 延迟执行,延迟到调用函数本身执行完毕之后再执行
   - 用于某些不论什么情况都要执行的操作,如关闭文件等,一般来说由于程序逻辑的局限性,在每一种分支中都要写,但是通过延迟执行可以极大简化这一过程,同样的适用于创建网络链接,关闭网络链接,开锁关锁这种成对出现的操作中
   - `defer`操作的匿名函数由于闭包的性质不仅可以访问到外部函数的局部变量,也可以访问到外部函数的返回结果
     ```Go
     func double (x int) (result int) {
         defer func() {fmt.Printf("double(%d) = %d\n", x, result)}() // 不要忘记这个括号
         return x + x
     }
     ```
     当然要求返回值是**具名的**
2. 如果有多个defer函数将按照顺序反向执行
### Panic
1. 不到万不得已不要使用`panic`,诸如文件不存在等问题应该被**优雅地**解决掉,而不是直接让程序崩溃

### Recover
This section intentionally left blank(狗头)