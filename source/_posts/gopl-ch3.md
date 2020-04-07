---
title: gopl-ch3
tags: Go
abbrlink: 9f2f5f69
categories:
  - Programming Language
  - Go
date: 2020-04-07 09:59:33
---


## Chapter 3
> Basic Date Types
1. basic types
2. aggregate types
   - arrays
   - structs
3. reference types
   - pointers
   - slices
   - maps
   - functions
   - channels
4. interface types


### Integers

1. `int`类型的细分
   - `int8` 以及对应的无符号版本`uint8`
   - `int16`
   - `int32`
   - `int64`
   `byte`是`unit8`的别名，`rune`是`int32`的别名. 虽然一般认为`int`是32位的，但是`int`和`int32`是两种类型，并且需要显式的转换
2. 模运算只能用于整数. 而对于负数来说结果的符号始终与被模数相同`-5%3`和`-5%-3`的结果都是`-2`
3. 除法运算当两个操作数均为整数时结果为整数(小数截断),否则为浮点数
4. `&^`运算符(AND NOT)与非
5. 关于无符号整数。一般**只用在**比特运算或者某些特定场景中，而**不会仅仅因为所表达的数据确实是非负的就使用**，考虑如下例子
   ```Go
   medals := []string{"gold", "silver", "bronze"}
   for i := len(medals) - 1; i >= 0; i-- {
       fmt.Println(medals[i])
   }
   ```
   其中数组的长度确实始终是非负的，但是`len()`函数依然返回的是有符号整数而不是无符号整数。如果`len()`函数返回的是无符号整数的话，那么`i`也将是无符号整数，那么`i >= 0`将永远为真，当`i=0`时执行`i--`将使得`i`变为`int`的最大值。
   总之为了避免种种类似这种错误，避免在不必要的地方使用无符号整数
<!--more-->

### Floating-Point Numbers
1. 关于`float32`和`float64`，几乎不使用`float32`，由于以下原因
   - `float32`一般来说会随着计算不断地积累误差
   - `float32`能够计算的最小不出现误差的整数其实也很小
     ```Go
     var f float32 = 16777216 // 1 << 24
     fmt.Println(f == f+1) // 为真
     ```
   `float32`可以表示大概6位的精确十进制，而`float64`为15位
2. 与`NaN`的比较始终返回`false`，所以不能直接与`NaN`比较来判断，而需要使用`math.IsNaN`
   ```Go
   nan := math.NaN()
   fmt.Prinln(nan == nan, nan < nan, nan > nan) // 均为false
   ```

### Complex Numbers
### Booleans
1. 没有从数值变量到布尔变量的隐式转换(vice versa)，所以需要手动写`if`来判断
   ```Go
   i := 0
   if b {
       i = 1
   }
   ```

### Strings
1. `string`类型的长度是由`byte`数来决定的,而不是字符数,`string`一般以`UTF-8`进行编码,非`ASCII`码字符可能会占用多个`byte`，同样的对`string`进行取下标操作所取得的也是`byte`
2. 对`string`使用`slice`操作(不知道是不是叫这个，就用Python里的叫法吧)，得到的是一个对应范围的`byte`所组成的**新的**字符串
3. 可以使用Python中的省略slice起点和终点的语法
4. 使用双引号
5. Go中的一个字符被称为`rune` (好像是废话)
6. 对int使用string强制类型转换并不是将其转化为对应文字的字符串，而是对应值的字符如
   ```Go
   fmt.Println(string(65)) // "A" 而不是"65"
   ```
   如果需要转化的话，使用`strconv`包
7. 使用`rune`切片来获得以**字符**分割的数据,如`r := []rune(s)`
8. 使用`Buufer.bytes`来构建字符串
9. 常用转换函数
   - `strconv.Itoa(123)`
   - `strconv.Atoi("123")`
   - `strconv.ParseInt("123", 10, 64)` 十进制,最多64bit

### Constants
1. 声明一系列const值的时候，会发生值的顺延
   ```Go
   const (
       a = 1
       b // 顺延a的值
       c = 2
       d // 顺延c的值
   )
   ```
2. 由上述性质产生了常量产生器(*Constant Generator*) `iota`
   ```Go
   type Weekday int
   const (
       Sunday Weekday = iota
       Monday
       Tuesday
       Wednesday
       Thursday
       Friday
       Saturday
   )
   ```
   其中`Sunday`为0,之后的依次顺延
   并且可以使用更加复杂的表达式,如`1 << (10 * iota)`,每次计算的时候也是将`iota`递增1
3. `Go`中的字面常量实际上是一种**无类型常量**(*Untyped Contants*), 其具有更高的精度(可以认为至少有256位)，并且有远超基本类型所能表示的数值范围和精度.如`math.Pi`. 当字面常量被赋值给某个基本类型的时候实际上进行了隐式的类型转换，这就要求目标类型能够装得下数据