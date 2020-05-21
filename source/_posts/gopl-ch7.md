---
title: gopl-ch7
tags: Go
categories:
  - Programming Language
  - Go
abbrlink: 98429b70
date: 2020-05-21 16:36:07
---


## Chapter 7
> interfaces

### Interfaces as Contracts

### Interface Types
1. `io`包中的一些`interface`
   ```go
   package io
   type Reader interface {
       Read(p []byte) (n int, err error)
   }
   type Closer interface {
       Close() error
   }
   ```
2. 与结构体一样，`interface`也可以进行`embedding`操作，从而简化
   ```go
   type ReadWriter interface {
       Reader
       Writer
   }
   type ReadWriteCloser interface {
       Reader
       Writer
       Closer
   }
   ```
   但与`struct`不同的是，进行`embedding`操作的顺序没有关系，`Reader`在前还是`Writer`并不会对生成的新接口有什么影响

### Interfaces Satisfaction
1. `interface`也有类似于Java中的`isA`的关系，即一个具体变量只要实现了相关的接口便可以赋值给对应的接口变量
   ```go
   var w io.Writer
   w = os.Stdout
   w = new(bytes.Buffer)

   var rwc io.ReadWriteCloser
   rwc = os.Stdout
   rwc = new(bytes.Buffer) // 报错，*bytes.Buffer并没有Close method
   ```
   等号的右边甚至也可以是接口
   ```go
   w = rwc
   rwc = w // 报错
   ```
2. `interface`不存在Java中的动态绑定的性质
   ```go
   var w io.Writer
   w = os.Stdout
   w.Write([]byte("hello")) // ok
   w.Close() // 报错
   ```
   其中`w`对应的接口`io.Write`只有`Write`方法，**即使实际类型`os.Stdout`有`Close`方法也无法进行调用**，也即接口变量值能够调用其对应的接口方法
3. 空接口
   要想将一个值赋值给一个接口类型的变量，那么只需要该值实现了接口中定义的方法即可，所以，空的接口对传入的值没有任何要求，也即可以**接收任何类型的值**
   ```go
   var any interface{}
   any = true
   any = 12.34
   any = "hello"
   any = map[string]int{"one": 1}
   // ...
   ```
4. 此处需要理解一下最开始提到的*satisfied implicitly*
   由于上述所说的，只要一个值的类型实现了一个接口所要求的方法，便可以将该值赋值给接口变量，如`rwc`可以赋值给`rw`，但是在一般的面向对象的语言中，一个类所实现的接口是*写死的*，就算实现了对应的方法，但是却没有明确写出`implement`某一个新的接口的话，依然无法将其值赋值给新的接口变量，这一点在使用第三方的包的时候尤为有用，因为无法对第三方的包中的类所实现的方法进行任意的修改，但是在go中就不存在这个问题，你可以任意的定义新的接口使得第三方的值可以进行赋值
   ```java
   public interface I1 {
       void method1();
   }
   public interface I2 {
       void method1();
       void method2();
   }
   public class A implements I2 {
       // ...
   }
   I2 i = new A(); // 没问题
   I1 i = new A(); // 编译出错，由于A并没有显式的implements I1，即使他实现了I1中所需的方法
   ```

### Parsing Flags with `flag.Value`

### Interface Values
1. `interface`的实际构成可以理解为两个部分
   - `type`
   - `value`
   其中，`type`指的是复制给接口变量的值的类型，如上述的`*bytes.Buffer`,而`value`则是指的具体的值，需要注意的是，当我恩声明一个新的interface变量但不进行赋值的时候，这个变量的`type`以及`value`都会被初始化为`nil`，但是也有可能出现`type`不为`nil`但是`value`为`nil`的情况，只需要赋值的时候，等号右边的变量为`nil`即可，这时的接口变量并不等于`nil`，换言之，将一个值为`nil`的变量赋值给接口变量并不会使接口变为`nil`
   > 感觉根本原因在于在go中的`nil`依然是有类型的?
2. `interface`*有可能*是可比的(*Comparable*)，比较两个`interface`实际上是比较`interface`的`value`，但是`value`本身却不一定可比。这与go中的一般的值不同，大多数的值类型是否可比是确定的，如基本类型是可比的，而`map`，`slice`是不可比的，但是由于interface是个究极缝合怪，可以承载不同的值，就导致其本身不一定可比，如果强行比较会发生`panic`

### Sorting with `sort.Interface`
1. 为想要排序的类型实现三个方法即可
   - `Less`
   - `Len`
   - `Swap`
   可以说自由度很高了，最佳实践是为每一种排序方式定义一个包装类型，分别定义这三个方法

### The `http.Handler` Interface
基本声明如下
```go
package http
type Handler interface {
    ServerHTTP(w ResponseWriter, r *Request)
}
func ListenAndServe(address string, h Handler) error
```

### The `error` Interface
基本声明如下
```go
type error interface {
    Error() string
}

func New(text string) error {return &errorString{test}}
type errorString struct {text string}
func(e *errorString) Error() string {return e.text}
```

### Example: Expression Evaluator

### Type Assertions
基本形式如下
```go
x.(T)
```
其中`x`是一个接口类型的表达式,`T`是某一种类型
1. 如果`T`是某一种具体类型,那么就会尝试将`x`中的`value`部分提取出来,也就是**运行时类型**,实际上就是用来将动态类型提取出来，因为前面说过，一个接口变狼只能够调用接口中规定的方法，而不能调用实际类型的中的方法
2. 如果`T`也是接口类型，并且`x`满足`T`，
3. 举例如下
   第一种
   ```go
   var w io.Writer // 接口类型的变量
   w = os.Stdout
   f := w.(*os.File) // 成功
   c := w.(*bytes.Buffer) // 失败
   ```
   第二种
   ```go
   var w io.Writer
   w = os.Stdout
   rw := w.(io.ReadWriter) // 成功
   w = new(ByteCounter)
   rw = w.(io.ReadWriter) // 失败
   ```
4. 一般情况下如果失败会发生`panic`,但是可以通过在赋值的左边加入一个标示`ok`来避免`panic`
   ```go
   if f, ok := w.(*os.File); ok {
       // ... 使用f
   }
   ```

### Discriminating Errors with Type Assertions
```go
import (
    "errors"
    "syscall"
)
var ErrNotExist = errors.New("file does not exist")

func IsNotExist(err error) bool {
    if pe, ok := err.(*PathError); ok {
        err = pe.Err
    }
    return err == syscall.ENOENT || err == ErrNotExist
}
```

### Querying Behaviors with Interface Type Assertions
### Type Switches
使用type switch可以非常方便的对某一个未知变量的不同可能类型采取不同操作
```go
switch x.(type) {
    case nil: // ...
    case int, unit: // ... 
    case bool: // ...
    case string: //...
    default: // ...
}
```
### Example: Token-Based XML Decoding
### A Few Words of Advice
1. 不要泛滥的使用接口类型，当一个接口类型被定义的时候，至少有是有两个具体类型能够*satisfy*才行
2. 不要为了面向对象而面向对象