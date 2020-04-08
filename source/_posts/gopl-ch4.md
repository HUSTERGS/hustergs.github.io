---
title: gopl-ch4
tags: Go
abbrlink: 14bcaca
categories:
  - Programming Language
  - Go
date: 2020-04-07 18:07:45
---

## Chapter 4
> Composite Types

### Arrays
> 由于数组是固定长度的，与切片(*slice*)相比有诸多不便，所以实际上很少用到，但是要理解切片就需要理解数组
1. 数组元素被自动初始化为零值
2. 数组字面量
   ```Go
   var q [3]int = [3]int{1,2,3}
   var r [3]int = [...]int{1,2}
   ```
   使用`...`填充数组长度，表示长度由字面量的个数决定(如果直接留空的话就是切片而不是数组)
3. 可以直接在字面量初始化的时候指定mouxie下表对应的值
   ```Go
   type Currency int
   const (
       USD Currency = iota
       EUR
       GBP
       RMB
   )
   symbol := [...]string{USD: "$", ....}
   ```
   如果没有指定所有下表对应的值,那么其他值将被初始化为零值
   ```Go
   r := [...]int{99: -1} // 长度为100,除了最后一个元素为-1之外,其余全部为0
   ```
4. 如果数组的元素本身是可比(*Comparable*)的话，那么数组也是可比的
5. 关于数组的传参。与大多数语言不同，数组作为参数传递的时候也是进行的**值传递**，也即每一个元素都被**复制**了一遍，即使会产生效率问题。当然也可以通过手动传递指针来解决
<!-- more  -->
### Slices
1. `slice`实际上是对Array的一种包装(个人理解),每一个`slice`包含以下几个成分
   - 一个指针(*pointer*),指向某一个数组
   - 长度(*length*), 指的是切片本身的长度,也即切片当前所包含的元素的个数 通过`len()`获得
   - 容量(*capacity*), 切片所能够容纳的最大元素,实际上就是其指向数组的长度 通过`cap()`获得
   多个slice可能指向同一个数组,如对切片再进行切片操作，得到的就是另外一个指向同一个数组的切片
2. 超过数组长度(也即容量)的切片会导致错误，而超过长度的切片则会自动拓展切片本身(**注意这个地方所说的切片操作是对已有的slice进行切片，而不是创建切片时对数组的切片**)
   ```Go
   months := [...]string{1: "January", /* ... */, 12: "December"}
   summer := months[6:9] // length = 3, cap = 13
   endlessSummer := summer[:5] // 超过了长度3，则summer的length会被拓展为5，也即months[6:11]
   ```
   由于切片中实际上包含了对源数组的引用，所以可以通过切片对数组进行修改
3. 与数组不同，**切片之间不能进行比较**，不能使用 `==` 来判断两个切片是否相等，唯一的特例是与`nil`的比较. 对于`[]bytes`类型来说，标准库提供了高度优化的`bytes.Equal`函数来进行比较，但是对于其他的类型来说，我们必须自己手动判断是否相等，尤其是在切片本身可以包含切片的条件下，对于`相等`所要判定的深度就会对切片本身是否相等产生影响。所以干脆就取消了切片之间的比较。
4. 切片的零值也是`nil`，没有对应的数组，所以`length`和`capacity`都为0，但是同时我们也应该注意到，存在长度和容量均为0但是不是`nil`的切片，如`s := []int`，或者我们也可以写成`s := int(nil)`，所以如果我们要判定一个切片是否为空(没有包含任何元素)，需要使用`len(s) == 0`而不是`s == nil`
5. 也可以使用`make`函数来创建`slice`
   ```Go
   make([]T, len)
   make([]T, len, cap) // same as make([]T, cap)[:len]
   ```
6. 使用`append`函数对切片进行拓展,实际上就是将新的值填入切片所指向的数组之中,如果空间不够的话则会创建新的数组并将原有的数组拷贝到新的数组中,当然,在这个过程中会使用一些技巧来避免反复拷贝,如将数组的长度变为两倍而不是每次增加一个
   ```Go
   runes = append(runes, r) // 注意不是直接改变,而是需要将返回值复制给拓展对象
   ```

### Map
1. 基本形式`map[K]V`,其中键`K`需要是可比的,也即可以使用`==`来操作
2. 同样可以使用`make`来构建一个`map`
   ```Go
   ages := make(map[string]int)
   // 等价于
   ages := map[string]int{}
   
   ages := map[string]int {
       "alice" : 31,
       "charlie": 34
   } // map字面量
   // 等价于
   ages := make(map[string]int)
   ages["alice"] = 31
   ages["charlie"] = 34 
   ```
3. 基本操作
   ```Go
   // 查询或赋值
   ages["alice"] = 32
   // 删除
   delete(ages, "alice")
   
   ```
   以上操作都是安全的(针对非`nil`的`map`来说)，即使所操作的键不在map中，查询的时候如果键不存在将会返回对应值类型的零值(但是似乎不会创建对应的键，仅仅是返回一个值)，所以可以有以下操作
   `ages["bob"]++`而不用担心`bob`不存在
4. 由于`map`的元素**不是一个变量**，所以我们无法取得他的地址，以下语句是错误的
   ```Go
   _ = &ages["bob"] // 编译错误
   ```
   其中一个原因是，在map中元素不断变化的过程当中，元素的地址可能会发生变化
5. 关于`map`的遍历
   ```Go
   // 遍历
   for name, age := range ages {
       fmt.Printf("%s\t%d\n", name, age)
   }
   ```
   是随机的，并且是**有意如此实现**，为了不让遍历的顺序成为某种依赖。如果想要按照某种顺序来遍历一个`map`，需要先将`key`排序，再通过遍历排序好的`key`来达到目的
   ```Go
   import "sort"

   var names []string
   for name := range ages {
       names = append(names, name)
   }
   sort.Strings(names)
   for _, name := range names {
       fmt.Printf("%s\t%d\n", name, ages[name])
   }
   // 可以通过预先分配数组来避免append操作，进而提高效率
   ```
6. 对于为`nil`的`map`的许多操作都是非法的，如插入
   ```Go
   var ages map[string]int
   ages["carol"] = 21 // 运行错误，由于ages是nil
   ```
7. 判断一个`key`是否在`map`中
   由于`map`的特性，使得不论键是否存在都会返回一个值，所以需要用其他的方法判断
   ```Go
   age, ok := ages["bob"]
   if !ok { /* .... */}
   // 或者更常用的
   if age, ok := ages["bob"]; !ok { /* ... */}
   ```
   索引操作返回的第二个值`ok`可以表示键是否存在
8. 与索引相同，我们也无法简单地使用`==`来判断两个`map`是否相等
9. 如果遇到某些不能比较的类型,想要将其作为map的键需要将其转化为可比较类型,常常是定义一个辅助函数,如将`slice`转化为`string`

### Structs
1. 例子
   ```Go
   type Employee struct {
       ID   int
       Name string
       Address  string
       DoB  time.Time
       Position string
       Salary   int
       ManagerID    int
   }
   var dilbert Employee
   ```
   可以将相同类型的成员写在一行中，如`Name, Address string`
2. 成员的顺序对于识别一个`struct`非常关键，顺序不同但是成员相同的两个结构依然是不同的，比如将`Name`和`Address`交换一下，那么就是一个新的结构
3. 一个结构不能有一个成员是他自己的类型(不是说值，而是类型，`Employee`结构之中不能有一个成员也是`Employee`)，但是允许成员是其指针，也即`Employee`可以有一个成员是`*Employee`类型，这就使得二叉树，链表等结构成为可能
4. 结构成员的大小写决定了他的可导出性(小写开头的成员只能在所在包内访问)
5. 结构字面量
   ```Go
   type Point struct{X, Y int}
   p := Point{1, 0}
   q := Point{X: 1} // 注意不是"X"
   ```
   第一种写法要求对于**每一个成员**都给出确定的值，由于这种写法对于代码编写者不友好，并且拓展性很差(如果后期`Point`新增了新的成员`Z`，则会报错)。而第二种写法就很棒，对于未写出的值将会设置为其类型对应的零值
6. 结构类型参数的传递同样是经过复制，所以常常使用指针来提高效率
7. 结构嵌入和匿名成员(*Struct Embedding and Anonymous Fields*)
   考虑如下两个结构
   ```Go
   type Circle struct {
       X, Y, Radius int
   }

   type Wheel struct {
       X, Y, Radius, Spokes int
   }
   var w Wheel
   w.X = 8
   w.Y = 8
   ...
   ```
   可以发现`Circle`和`Wheel`有相同的成员，于是我们可能会想到进行复用
   ```Go
   type Point struct {
       X, Y int
   }
   type Circle struct {
       Center Point
       Radius int
   }
   type Wheel struct {
       Circle Circle
       Spokes int
   }
   var w Wheel
   w.Circle.Center.X = 8
   w.Circle.Center.Y = 8
   ...
   ```
   但是会发现获取成员变得非常麻烦，要经过3个`.`操作。这时就可以使用结构嵌入的办法定义匿名成员来解决这个问题
   ```Go
   type Circle struct {
       Point // 没有名字
       Radius int
   }

   type Wheel struct {
       Circle // 没有名字
       Spokes int
   }

   var w Wheel
   w.X = 8 // 等价于w.Circle.Point.X = 8
   w.Y = 8
   ```
   实际上我们会发现所谓*匿名*，其实并不匿名，我们依然可以通过类型来访问到对应的成员，成员名即为所对应的类型名.在获得一方面的方便时，这种类型结构的字面量会变得很复杂
   ```Go
   w = Wheel{Circle{Point{8, 8}, 5}, 20}
   // 或者
   w = Wheel {
       Circle: Circle {
           Point: Point {
               X: 8,
               Y: 8 // 这个地方没有逗号
           },
           Radius: 5, // 这个地方的逗号也是必要的
       },
       Spokes: 20, // 注意这个地方的逗号是必要的
   }
   ```
   由于匿名变量实际上有名字，所以不能够有两个相同类型的匿名成员，同样类型也会影响其导出型，因为成员名即为类型名
   总的来说这种操作实际上是一种语法糖，后面我们会看到，任何具名类型或者指向具名类型的指针都能够作为匿名成员。而不仅仅是结构，那么为什么我们需要对可能没有成员的类型也提供支持呢，原因是匿名成员的**方法**(*method*)也会被嵌入，这是在Go中实现面向对象思想的重要途径

### JSON
1. 例子
   ```Go
   type Movie struct {
       Title string
       Year int `json:"released"`
       Color bool `json:"color,emitempty"`
       Actors []string
   }
   ```
   所有结构成员为大写，通过在在成员之后添加*标记*，来修改最终呈现的`JSON`中的键名，可以额外使用`emitempty`将零值去除(此处为`false`)
2. 使用
   - `json.Marshal`
   - `json.MarshalIndent`
   - `json.Unmarshal`
   来进行基本的转化
   以及`json.Encoder`和`json.Encoder`来处理流式数据

### Text and HTML Templates
博大精深，先放着(狗头)