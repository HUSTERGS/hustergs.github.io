---
title: gopl-ch6
tags: Go
categories:
  - Programming Language
  - Go
abbrlink: ef45abe6
date: 2020-04-09 10:33:16
---


## Chapter 6
> Methods

### Methods Declarations
1. 在函数名前面加上对应的具名类型即可
   ```Go
   package geometry
   import "math"

   type Point struct{X, Y float64}
   func Distance(p, q Point) float64 {
       return math.Hypot(q.X-p.X, q.Y-p.Y)
   }
   // 方法
   func (p Point) Distance(q Point) float64 {
       return math.Hypot(q.X-p.X, q.Y-p.Y)
   }

   p := Point{1, 2}
   q := Point{4, 6}
   fmt.Println(Distance(p, q))
   fmt.Println(p.Distance(q))
   ```
   额外的`(p Point)`称之为方法的`接收者`(*receiver*)
   并不是非要使用`this`或者`self`,一般采用对应类型的开头字母的小写(`p`对应`Point`)
   两个`Distance`函数并不会产生名称冲突,他们一个是包级别的函数(`geometry.Distance`),另一个是具名类型`Point`的方法(`Point.Distance`)
   > Go中的`struct`**不支持重载**,所以不能有两个方法名相同,也不能有成员和方法相同
<!-- more -->
2. `方法`是针对*具名类型*来说的,并不是仅仅局限于`struct`,由于我们可以很方便的为几乎所有类型定义别名,所以我们也可以很方便的对各种类型进行包装,如`int`,`map`(无法对实际上是`interface`或者`指针`的具名类型定义方法)

### Methods with a Pointer Receiver
1. 由于参数传递的时候是传递的拷贝，包括使用方法的时候接收者的传递,所以当
   - 接收者很大，包含的成员很大
   - 需要对接受者本身进行修改
   时，就需要将方法的接收者类型改为指针
   ```Go
   func (p *Point) ScaleBy(factor float64) {
       p.X *= factor
       p.Y *= factor
   }
   ```
   指向具名类型的指针和具名类型本身是唯二两种可以杂方法前的那一坨中使用的类型
   > 在实际的编程中,如果出现一个方法的接收者类型是指针,那么说明所有的方法都是接收指针的
   指针的类型的方法有以下几种方式调用,均是有效的
   ```Go
   r := &Point{1, 2} // 不是说不能获得一个字面量的地址吗?
   r.ScaleBy(2)
   fmt.Println(*r)

   p := Point{1, 2}
   pptr := &p
   pptr.ScaleBy(2)
   fmt.Println(p)

   p := Point{1,2}
   (&p).ScaleBy(2)
   fmt.Println(p)
   ```
   可以发现是真的丑,所以编译器帮我们做了一些处理,当p是一个具名类型的变量时,`p.MethodName`会被自动替换为`(&p).MethodName`,所以我们可以愉快的写出`p.ScaleBy(2)`而不用担心类型不匹配的问题
   同时我们也应该要意识到,这个方法只能用于`变量`,即那些可以通过`&`取得地址的变量,直接对字面量使用该方法会出错
   ```Go
   Point{1, 2}.ScaleBy(2) // 编译错误
   ```
   > 其实有点不懂,因为像上面初始化的时候不就出现了`r := &Point{1,2}`吗,书上给的解释是说*无法获得临时值的地址*，而且实际实验的时候`Point{1,2}.ScaleBy(1)`确实会报错，但是`(&Point{1,2,3}).ScaleBy(1)`就没有报错，说明确实是可以做到的吧
   以下做一个总结,针对实际调用的对象和调用的方法中所指定的接收者类型有
   - 实际调用为具名类型,接收者为具名类型
     直接调用,没有问题
   - 实际调用为具名类型,接收者为具名类型的指针
     将`v.MethodName`隐式转化为`(&v).MethodName`
   - 实际调用为具名类型的指针,接收者为具名类型
     将`v.MethodName`隐式转化为`(*v).MethodName`
   - 实际调用为具名类型的指针,接收者为具名类型的指针
     直接调用,没有问题
   > 在上述第二种第三种情况中,虽然编译器会帮我们添加,但是在非直接调用的时候,如`fmt.Println(p)`间接调用类型p中的`String`方法的时候,如果String方法的接收者是指针,但是p是一个实例,这时就不会自动转换,需要手动写成`fmt.Println(&p)`
2. `nil`可以作为方法的接收者
   与其他的大部分语言不同，如其他语言中的`None`, `null`, `nullptr`，均无法调用其任何方法，就是纯粹的空值，但是在Go中允许nil作为方法的接收者，尤其是当`nil`还是某一个类型的零值的时候(说到底感觉其实是因为具名类型的方法虽然有所谓*接收者*的说法，但是实际上还是与类型而不是具体的实例绑定的，感觉`v.MethodName(parameter-list)`与`TypeName.MethodName(v, parameter-list)`是等价的，所以即使是`nil`也只是作为参数传入函数)

### Composing Types by Struct Embedding
1. 可以在结构中嵌入匿名成员,不仅仅是结构,而是所有的具名类型(该具名类型本身不能为指针或者接口)以及指向具名类型的指针,嵌入类型的方法和成员都将被嵌入到被嵌入者中
2. 方法原本只能在具名类型中定义,但是由于嵌入的存在,匿名结构也可能具有方法,并且非常有用,考虑如下例子
   ```Go
   // 原版
   var (
       mu sync.Mutex
       mapping = make(map[string]string)
   )

   func Lookup(key string) string {
       mu.Lock()
       v := mapping[key]
       mu.Unlock()
       return v
   }
   ```
   通过使用匿名结构可以将两者粘合在一起
   ```Go
   var cache = struct {
       sync.Mutex
       mapping map[string]string
   } {
       mapping: make(map[string]string),
   }

   func Lookup(key string) string {
       cache.Lock()
       v := cache.mapping[key]
       cache.Unlock()
       return v
   }
   ```
### Method Values and Expression
1. 方法可以由两种方式导出
   - 通过`Instance.MethodName`方式导出,如
     
     ```Go
     p := Point{1,2}
     distanceFromP := p.Distance // distanceFromP是一个函数，绑定了实例p
     fmt.Println(distanceFromP(Point{4, 6}))
     ```
     有点函数闭包的感觉
   - 通过结构名导出,这种方式导出之后,原有的方法会新加入一个参数用于确定*接收者*
     ```Go
     distance = Point.Distance
     fmt.Println(distance(p, q))
     ```
   这两种方法就有点像是实例方法的导出和类方法(静态)的导出

### Encapsulation
1. Go中也有所谓`setter`和`getter`,但是为了简洁,一般`getter`(以及类似的`Lookup`,`Find`)都不会加上前缀.

