---
title: gopl-ch2
tags: Go
categories:
  - Programming Language
  - Go
abbrlink: e8286fff
date: 2020-04-06 20:40:53
---


## The Go Programming Language
> 本系列将从《The Go Programming Language》的第二章开始，记录一些阅读过程中遇到的值得记录的知识点，也希望可以坚持下去，读完整本书

## Chapter 2 
> Program Structure

### Names
1. 在函数外定义的实体(函数本身或者变量),会根据其首字母大小来决定其访问权限，小写字母开头的实体只能在包内访问，大写字母开头的会被`export`，也即其他包中可以通过`import`的方式访问到，如`fmt.Printf`。包名本身始终是小写的
2. 使用驼峰式而不是下划线

### Declarations
1. 四种主要的声明
   - var
   - const
   - type
   - func

2. 函数没有`void`返回选项，如果什么都不返回，直接为空即可

### Variables
1. 使用`var`来声明变量
   ```Go
   var name type = expression
   ```
   可以省略类型`type`或者不赋值，省略右侧的`expression`，省略类型会进行自动的类型推断，省略表达式会自动将变量赋值为其类型所对应的零值,在go中没有所谓的**未初始化的变量**
2. 可以在一条声明语句中初始化多个不同类型的变量
   ```Go
   var i, j, k int
   var b, f, s = true, 2.3, "four"
   ```
<!--more-->

3. 使用简写声明(`short variable declaration`)来**声明**或者**赋值**局部变量
   ```Go
   anim := gif.GIF{LoopCount: nframes}
   freq := rand.Float64() * 3.0
   ```
   局部变量**基本上**都是使用这种方法进行声明, 只有在某些情况下才需要使用`var`(A `var` declaration tends to be reserved for local variables that need an explicit type that differs from that of the initializer expression, or for when the variable will be assigned a value later and its inital value is unimportant)如
   ```Go
   i := 100
   var boiling float64 = 100 // 类型与右侧不一致
   var names []string // 仅仅是声明
   var err error
   var p Point
   ```
   同时注意`:=`是声明,而`=`是赋值,Go中有类似于Python中的语法,称为`tuple assignment`
   ```Go
   i, j = j, i // 交换i和j的值
   f, err := os.Open(name) // 此处注意和上面的区别
   ```
   使用`:=`的时候,左侧的不一定非要是新的变量,如果左侧的变量名在之前出现过,那么对该变量的操作就会是赋值,而不是新建一个新的同名变量.但**至少要保证**有一个**新的变量**被声明
   
   ```Go
   f, err := os.Open(infile) // 声明了新的变量f和err
   // ...
   f, err := os.Creat(outfile) // 编译错误,此时f和err都已经出现过,所以没有新的变量出现,应该使用`=`而不是`:=`

   out, err := os.Create(outfile) // 正确,其中out为新声明的变量,而对err是进行了赋值操作
   ```

4. 指向函数内局部变量的指针
   在Go中,可以放心的返回指向函数内部变量的指针,而不必担心其指向的对象被销毁
   ```Go
   var p = f()
   func f() *int {
       v := 1
       return &v
   } // 没毛病
   fmt.Println(f() == f()) // false
   // 每一次调用会产生新的值
   ```
   > 此处没有讲具体的实现细节

5. 使用`new`函数来创建指针变量
   注意`new`是**函数**不是关键字(所以可以被覆盖),使用`new`可以创建一个无名的变量并返回指向其的指针
   ```Go
   p := new(int) 
   /*等价于
   var unname int
   p := &unname
   */
   fmt.Println(*p) // 0
   ```
   使用`new`函数通常来说得到的指针应该指向不同的地址,但是对于`struct{}`或者`[0]int`,依赖于编译器的具体实现,可能会产生相同的地址
   总的来说用到`new`函数的机会其实很少,主要是因为,最常用的无名变量只有`struct`,而对于`struct`有更为方便的字面量来表示.

### Assignments
1. 元组赋值(`tuple assignment`)
   ```Go
   func gcd(x, y int) int {
       for y != 0 {
           x, y = x, x%y // 元组赋值
       }
       return x
   }
   ```
   很多函数都会返回元组,常常是两个值,一个是返回的数据,另一个表示错误. 同时在
   - map查找(*map lookup*)
   - 类型推断(*type assertion*)
   - 管道接收(*channel receive*)
   中也会返回两个值
   ```Go
   v, ok = m[key]
   v, ok = x.(T)
   v, ok = <-ch
   ```
   使用`_`来丢弃不需要的值
2. 使用等号进行比较的两个变量之间必须是*可赋值的*(`assignable`), 如`nil`可以赋值给所有的引用变量或者接口

### Type Declarations
1. 为某种类型提供一个*别名*, 类似于C中的`typedef`，可以更方便的表达语义，如`int`可以是秒数也可以是日期。这种*别名*常常用在包一级别中，如以下的例子
   ```Go
   package tempconv
   import "fmt"

   type Celsius float64
   type Fahrenheit float64

   const (
       AbsoluteZeroC Celsius = -273.15
       FreezingC Celsius = 0
       BoilingC Celsius = 100
   )
   func CToF(c Celsius) Fahrenheit { return Fahrenheit(c * 9/5 + 32) }
   func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9)}
   ```
   注意，虽然`Celsius`和`Fahrenheit`本质上都是`float64`类型，但是依然认定为两个不同的类型，这两个类型之间不能进行比较或者在同一个数学表达式中使用。
2. 在定义一个类型之后可以为其编写特定的方法，称为*the type's methods*

### Packages and Files
1. 包的初始化
   包在初始化是会按照顺序执行包级别的变量初始化(*initializing package-level variables*)。而包本身的加载顺序则是通过go本身先对`.go`文件进行排序决定的。对于一些简单的变量来说这样做可能没什么问题，但是对于比较复杂的情况常常使用初始化函数(*init function*)，而不是零零散散的变量声明.
   ```Go
   func init() { /* ... */}
   ```
   `init`函数不能被调用或者引用，在包被导入的时候，`init`函数会自动执行以进行初始化(init函数里面的变量应该还是作为局部变量而不是包级变量，也即init函数只能进行初始化相关工作，而不能进行变量的声明,我猜)

### Scope
