---
title: python-cookbook-ch4
tags: Python
categories:
  - Programming Language
  - Python
abbrlink: 65f283cf
date: 2020-07-19 23:31:59
---

**终于考完了，开始更新啦！**
## Chapter 4
> Iterators and Generators
> 这一章的内容个人其实很少用到，但是又感觉还是比较重要，所以可能记录的点比较多，甚至于全部记录

1. 手动消费一个迭代器(*iterator*)
    使用`next(iterator[, default])`函数
    ```py
    with open('/etc/passwd') as f:
        try:
            while True:
                line = next(f)
                print(line, end='')
        except StopIteration:
            pass
    ## 或者
    with open('/etc/passwd') as f:
        while True:
                line = next(f, None)
                if line is None:
                    break
                print(line, end='')
    ```
    也即，当传入第二个参数的时候，将会返回第二个参数作为默认值，而不是抛出`StopIteration`异常

<!-- more -->

2. 通过在自定义类中实现`__iter__`方法来使其可以作为迭代器被使用(类似于Java中的`Iterator`接口需要实现`hasNext`以及`next`函数一样)。需要注意的是，`__iter__`返回的需要是一个实现了`__next__`方法的对象

3. 通过在函数中使用`yield`来实现一个自定义的迭代模式（跳过）
4. `__reversed__`方法使得可以通过`reversed(iterator)`的方式来直接倒序遍历某一个迭代器，如果没有实现`__reversed__`方法就只能手动将迭代器转化为list再倒序遍历，而这将有可能耗费大量的内容
5. 迭代器的切片
   使用`itertools.islice(iterator, startIndex, stopIndex)`来得到迭代器的切片。需要注意的是，该函数会**使用**迭代器，而迭代器是无法逆转的，如果介意这一点的话，可以考虑先转成`list`

6. 根据条件消耗掉迭代器的前n个元素
   ```py
   from itertools import dropwhile
   with open('/etc/passwd') as f:
       for line in dropwhile(lamdba line: line.startswith('#'), f):
           print(line, end='')
   ```
   用于去除文件中开头的所有注释
   > 注意`dropwhile`遇到不满足的条件的即会跳出而`(line for line in f if not line.startswith('#'))`则会去除所有的注释，而不只是开头的。

7. 排列组合
   使用`itertools.permutations`、`itertools.conbination`、`itertools.conbination_with_replacement`
   分别为全排列，全组合，以及可以重复的全组合
   第一个参数均为可遍历对象，第二个可选参数为选择的元素的个数
   如
   ```py
   from itertools import conbination_with_replacement
   items = ['a', 'b', 'c']
   print(conbination_with_replacement(items, 2))
   # [('a', 'a'), ('a', 'b'), ('a', 'c'), ('b', 'c'), ('b', 'b'), ('c', 'c')]
   ```

8. 使用`zip`来同时遍历两个等长数组
   ```py
   a = list(range(10))
   b = list(reversed(range(10)))
   for x, y in zip(a, b):
       print(x, y)
   ```

9. 拼接遍历两个**容器**
    使用`itertools.chain`，使用方法非常简单
    ```py
    from itertools import chain
    a = [1,2,3.4]
    b = set(['a', 'b', 'c'])
    for x in chain(a, b):
        print(x)
    ```
    **注意**: 
    `chain`的特殊之处在于
    1. 比起`a + b`来说效率更高，由于两个`list`相加会创建一个新的`list`，但是`chain`并不是(此处假设`a`, `b`为`list`)
    2. 可以对不同类型进行拼接，如上述所示的，一个是`tuple`，而另一个是`list`
    同时，`chain`并不会对容器的特性进行整合，如在两个`set`上进行`chain`操作时，并不会对两个`set`中相同元素进行去重

10. **flattening**一个嵌套的序列
    ```py
    from collections import Iterable

    def flatten(items, ignore_types=(str, bytes)):
        for x in items:
            if isinstance(x, Iterable) and not isinstance(x, ignore_types):
                yield from flatten(x)
            else:
                yield x
    for x in flatten(items):
        print(x)
    ```
    其中:
    - `ignore_types`用于过滤一些可迭代对象，如`str`以及`bytes`
    - `yield from`用于在一个生成器函数中调用另一个生成器函数

11. 合并两个已排序序列(归并)
    ```py
    import heapq
    a = [1,4,7,10]
    b = [2,5,6,11]
    for x in heapq.merge(a, b):
        print(x)
    # 1,2,4,5,6,7,10,11
    ```
    **注意**:
    - `merge`并不会检查两个序列是否已经排好序
    - `merge`也不会生成一个中间对象，而是返回一个迭代器，所以效率很高
