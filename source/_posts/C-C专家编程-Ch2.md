---
title: C-C专家编程 Ch2
tags: C
categories:
  - Programming Language
  - C
abbrlink: b44e0c73
date: 2020-09-16 16:55:58
---


1. 一般来说只要看到`malloc(strlen(str))`基本都是错误的，正确的应该是`malloc(strlen(str)+1)`，由于需要为`\0`符号留出空间
2. `NUL`与`NULL`，前者为`\0`，后者表示什么也不指向（空指针）
<!-- more  -->
3. 多行字符串会发生自动拼接
   ```c
   printf("First line "
        "Second line "
        "Third line");
   ```
   将会打印`First line Second line Third line`，其中除了最后一个字符串之外，其余字符串末尾的`\0`均被省略
   以往通过`\`来进行多行信息拼接的方法不再需要。
   同时需要注意的是这种默认行为在字符串数组的声明中可能带来的一些问题
   ```c
   char *available_resources[] = {
       "color monitor",
       "big disk",
       "Cary" // 此处少了一个逗号
       "on-line drawing routhines",
       "mouse",
   };
   ```
   缺少一个逗号将会使两个元素自动进行拼接，不仅仅字符串的值会发生变化，数组的大小也不再是预期值
<!-- more -->
4. 奇技淫巧
   ```c
   generate_initializer(char * string)
   {
       static char separator = ' ';
       printf("%c %s \n", seperator, string);
       seperator = ',';
   }
   ```
   通过`static`变量来消除打印一个列表时想要在对象之间使用逗号之类的分隔符但是却要处理最后一个多余分隔符的问题
5. C中由于一些符号的重载导致了很多理解上的问题
   ```c
   p = N * sizeof * q; // 此处会让人疑惑到底是一个乘号还是两个，其实是一个，当sizeof的对象为类型时，需要加上括号，当是一个变量的时候就不必加括号。
   apple = sizeof(int) *p; // 到底是int的长度乘上p还是将p的值解引用之后的值强制类型转换为int之后再求其size？经过测试之后可以发现是前者
   ```

6. 一些容易误解的优先级问题
   - `*p.f`
     容易误解为`(*p).f`，`.`的优先级要高于`*`，所以有了`->`操作符来消除这个问题
   - `int *ap[]`
     `ap`是一个指针数组，而不是一个指向int数组的指针，`[]`的优先级要高于`*`
     > 如果没有记错的话，面对此类关于声明的时候的类型问题时，可以直接将声明当作实际使用时所需要遵循的格式，也即使用ap的时候应该是类似于`*ap[index]`再结合`[]`的优先级要高于`*`就可以很容易理解`ap`是一个指针数组
   - `(val & mask != 0)`
     `!=`以及`==`的优先级要高于位运算符，所以实际上是`val & (mask != 0)`
   - `c = getchar() != EOF`
      `==`以及`!=`高于`=`
   - `msb << 4 + lsb`
     算术运算符高于位移运算符
   - `i = 1,2`
     逗号运算符在所有的运算符中优先级最低，所以虽然逗号逗号运算符的结果就是最右边操作数的值，但是这里`i`依然等于1

   对于不确定的地方可以直接加括号保证顺序

7. `x = f() + g() + h();`中并不能保证三个函数的求值顺序
8. 对于一个函数需要返回一个字符串或者某种数组之类的值时，有以下几种方法
    - 直接返回一个指向字符串常量的指针
      ```c
      char * func() {
        
      }
      ```
    - 使用全局声明的数组
      ```c
      char * func() {
          global_array[i] = ...;
          return global_array;
      }
      ```
    - 使用静态数组
      ```c
      char * func() {
          static char buffer[20];
          ...
          return buffer;
      }
      ```
      函数的下一次调用将会覆盖内容，需要手动进行数据的备份
    - 显式的分配内存
      ```c
      char * func() {
          char * s = malloc(120);

          return s;
      }
      ```
      不会由于重复调用而覆盖之前的数据。但是由于内存的分配和释放并不在同一个地点（一个在函数内部，一个在调用者），所以可能会出现忘记释放内存的情况
    - **调用者来分配/释放内存，并作为参数进行传递**
      ```c
      void func(char * result, int size){
          strncpy(result, "Test", size);
      }

      buffer = malloc(size);
      func(buffer, size);
      ...
      free(buffer);
      ```
      可以说是最好的解决方案