---
title: Python Standard
tags: Python
categories:
  - Programming Language
  - Python
abbrlink: 4b301912
date: 2019-08-11 00:06:08
---

# Python Standard

## Python 入门

1. 字符串乘法

   ```python
   3 * '123' = '123123123'
   ```

2. 三引号用于换行字符串

   ```python
   print("""\
   Usage: thingy [OPTIONS]
        -h                        Display this usage message
        -H hostname               Hostname to connect to
   """)
   ```

3. `_`在交互模式下表示最后打印的值

4. 数组切片为***浅拷贝***

5. `print`使用`end`参数来定义结束符

6. 方便的使用切片来遍历,而不是直接使用原数组

   ```python
   for w in words[:]:
   	pass
   ```

7. 全局变量和外层函数的变量不能在函数内部被直接赋值, 除非使用`global`等关键字

<!--more-->


8. 默认值是在 *定义过程* 中在函数定义处计算的

   ```python
   i = 5
   
   def f(arg=i):
       print(arg)
   
   i = 6
   f()
   ```

   打印的是`5`

   默认值只会执行一次。这条规则在默认值为可变对象（列表、字典以及大多数类实例）时很重要。比如，下面的函数会存储在后续调用中传递给它的参数:

   ```python
   def f(a, L=[]):
       L.append(a)
       return L
   
   print(f(1))
   print(f(2))
   print(f(3))
   ```

   这将打印出

   ```python
   [1]
   [1, 2]
   [1, 2, 3]
   ```
9. 使用`*`和`**`来解包元组和字典

10. 使用函数标注来标注类型信息

    ```python
    def somefunc(a : str, b : int) -> str:
        return 'Yes'
    ```

11. 将列表作为堆栈使用问题不大，由于列表本身就是在尾部的操作很方便，但是如果将其作为队列使用就会很慢，尤其是对于列表开头的元素，这种情况下使用`collections.deque`

    ```python
    from collections import deque
    queue = deque(["Eric", "John", "Michael"])
    queue.append("Terry")           # Terry arrives
    queue.append("Graham")          # Graham arrives
    queue.popleft()                 # The first to arrive now leaves
    # 'Eric'
    queue.popleft()                 # The second to arrive now leaves
    # 'John'
    queue                           # Remaining queue in order of arrival
    # deque(['Michael', 'Terry', 'Graham'])
    ```

12. `del`可以用于根据索引来清除列表中的某一个元素而不是根据值，也可以用于移除切片或者清空整个列表（由于切片是浅拷贝）

13. 要创建一个空集合只能用`set()`而不能用`{}`，后者是创建一个空字典

14. 当同时在多个序列中循环的时候使用`zip()`来将其内部的元素一一匹配

     ```python
     questions = ['name', 'quest', 'favorite color']
     answers = ['lancelot', 'the holy grail', 'blue']
     
     for q, a in zip(questions, answers):
     	print(q, a)
     ```

15. 比较操作可以传递？？？

    `-3 < -2 < -1`可以正常运行


16. 要使用 格式字字符串字面值 ，请在字符串的开始引号或三引号之前加上一个 `f` 或 `F` 。在此字符串中，你可以在 `{` 和 `}` 字符之间写可以引用的变量或字面值的 Python 表达式。  

    ```python
    year = 2016
    event = 'Referendum'
    f'Result of the {year} {event}'
    ```

17. 不懂为什么

    ![1564809743482](/home/gs/.config/Typora/typora-user-images/1564809743482.png)

18. 类中的私有变量实际山并不存在，而总是以下划线`_`开头约定为私有变量，或者

    由于存在对于类私有成员的有效使用场景（例如避免名称与子类所定义的名称相冲突），因此存在对此种机制的有限支持，称为 *名称改写*。 任何形式为 `__spam` 的标识符（至少带有两个前缀下划线，至多一个后缀下划线）的文本将被替换为 `_classname__spam`，其中 `classname` 为去除了前缀下划线的当前类名称。 这种改写不考虑标识符的句法位置，只要它出现在类定义内部就会进行。

    名称改写有助于让子类重载方法而不破坏类内方法调用

19. 生成器表达式，类似于生成列表和生成set，只是变为了简单的括号，相比等效的列表推导式更加**节省内存**

    ```python
    sum(i*i for i in range(10))
    ```

20. 对于日常文件和目录管理任务， `shutil` 模块提供了更易于使用的更高级别的接口:

    ```python
    import shutil
    shutil.copyfile('data.db', 'archive.db')
    shutil.move('/build/executables', 'installdir')
    ```

21. `glob` 模块提供了一个在目录中使用通配符搜索创建文件列表的函数:

    ```python
    import glob
    glob.glob('*.py')
    # ['primes.py', 'random.py', 'quote.py']
    ```

22. 终止脚本的最直接的方法是`sys.exit()`

23. `ramdom`提供了进行随机选择的工具

    `random.sample(range(100), 10)   # sampling without replacement`

24. 常见的数据存档和压缩格式由模块直接支持，包括：`zlib`, `gzip`, `bz2`, `lzma`, `zipfile` 和 `tarfile`。

25. 性能测量

    一些Python用户对了解同一问题的不同方法的相对性能产生了浓厚的兴趣。 Python提供了一种可以立即回答这些问题的测量工具。

    例如，元组封包和拆包功能相比传统的交换参数可能更具吸引力。timeit 模块可以快速演示在运行效率方面一定的优势:

    ```python
    from timeit import Timer
    Timer('t=a; a=b; b=t', 'a=1; b=2').timeit()
    # 0.57535828626024577
    Timer('a,b = b,a', 'a=1; b=2').timeit()
    # 0.54962537085770791
    ```

    与 `timeit` 的精细粒度级别相反， `profile` 和 `pstats` 模块提供了用于在较大的代码块中识别时间关键部分的工具。

26. 尽管这些工具非常强大，但微小的设计错误却可以导致一些难以复现的问题。因此，实现多任务协作的首选方法是将对资源的所有请求集中到一个线程中，然后使用 queue 模块向该线程供应来自其他线程的请求。应用程序使用 Queue 对象进行线程间通信和协调，更易于设计，更易读，更可靠。

27. `logging` 模块提供功能齐全且灵活的日志记录系统。在最简单的情况下，日志消息被发送到文件或 `sys.stderr`

    ```python
    import logging
    logging.debug('Debugging information')
    logging.info('Informational message')
    logging.warning('Warning:config file %s not found', 'server.conf')
    logging.error('Error occurred')
    logging.critical('Critical error -- shutting down')
    ```

    输出

    ```
    WARNING:root:Warning:config file server.conf not found
    ERROR:root:Error occurred
    CRITICAL:root:Critical error -- shutting down
    ```

28. Python 会自动进行内存管理（对大多数对象进行引用计数并使用 garbage collection 来清除循环引用）。 当某个对象的最后一个引用被移除后不久就会释放其所占用的内存。

    此方式对大多数应用来说都适用，但偶尔也必须在对象持续被其他对象所使用时跟踪它们。 不幸的是，跟踪它们将创建一个会令其永久化的引用。 weakref 模块提供的工具可以不必创建引用就能跟踪对象。 当对象不再需要时，它将自动从一个弱引用表中被移除，并为弱引用对象触发一个回调。 **典型应用包括对创建开销较大的对象进行缓存**

29. `array` 模块提供了一种 `array()` 对象，它类似于列表，**但只能存储类型一致的数据且存储密集更高**.

30. `deciaml`模块提供了运算所需要的足够精度

## 安装和使用Python

无



## Python语言参考

### 3. 数据模型

* 每个对象都有各自的编号、类型和值。一个对象被创建后，它的 *编号* 就绝不会改变
* `is` 运算符可以比较两个对象的编号是否相同；`id()` 函数能返回一个代表其编号的整型数。
* CPython 中，`id(x)` 就是存放 `x` 的内存的地址。
* 类型会影响对象行为的几乎所有方面。甚至对象编号的重要性也在某种程度上受到影响: 对于不可变类型，会得出新值的运算实际上会返回对相同类型和取值的任一现有对象的引用，而对于可变类型来说这是不允许的。例如在 `a = 1; b = 1` 之后，`a` 和 `b` 可能会也可能不会指向同一个值为一的对象，这取决于具体实现，但是在 `c = []; d = []` 之后，`c` 和 `d` 保证会指向两个不同、单独的新建空列表。(请注意 `c = d = []` 则是将同一个对象赋值给 `c` 和 `d`。)



## Python 标准库

### 内置函数

* `all()`, `any()` 参数为可迭代对象, `与`, `或`

* `chr()`返回 Unicode 码位为整数 i 的字符的字符串格式。

* `divmod(a, b)` 它将两个（非复数）数字作为实参，并在执行整数除法时返回一对商和余数。

* `enumerate`(*iterable*, *start=0*) 

* `globals`()返回表示当前全局符号表的字典。这总是当前模块的字典（在函数或方法中，不是调用它的模块，而是定义它的模块）。

* `bin()`, `hex()`, `oct()`将整数转化为不同进制的字符串

* `map`(*function*, *iterable*, *...*)返回一个将 *function* 应用于 *iterable* 中每一项并输出其结果的迭代器。

* `ord`(*c*)对表示单个 Unicode 字符的字符串，返回代表它 Unicode 码点的整数。例如 `ord('a')` 返回整数 `97`， `ord('€')` （欧元符号）返回 `8364` 。这是 [`chr()`](https://docs.python.org/zh-cn/3.7/library/functions.html#chr) 的逆函数。

* `sorted`(*iterable*, *, *key=None*, *reverse=False*)

* `zip`(**iterables*)返回一个元组的迭代器，其中的第 *i* 个元组包含来自每个参数序列或可迭代对象的第 *i* 个元素。 当所输入可迭代对象中最短的一个被耗尽时，迭代器将停止迭代。 当只有一个可迭代对象参数时，它将返回一个单元组的迭代器。 不带参数时，它将返回一个空迭代器。

  zip() 与 * 运算符相结合可以用来拆解一个列表:

  ```python
  x = [1, 2, 3]
  y = [4, 5, 6]
  zipped = zip(x, y)
  list(zipped)
  # [(1, 4), (2, 5), (3, 6)]
  x2, y2 = zip(*zip(x, y))
  x == list(x2) and y == list(y2)
  # True
  
  ```



### 内置类型

* `x % y`

* `x // y`

* 比较运算可以任意串连；例如，`x < y <= z` 等价于 `x < y and y <= z`，前者的不同之处在于 *y* 只被求值一次（但在两种情况下当 `x < y` 结果为假值时 *z* 都不会被求值）。

* `float.as_integer_ratio()`返回一对整数，其比率正好等于原浮点数并且分母为正数。 

* `float.is_integer()` 如果 float 实例可用有限位整数表示则返回 `True`，否则返回 `False`

* `str.``count`(*sub*[, *start*[, *end*]])

  反回子字符串 *sub* 在 [*start*, *end*] 范围内非重叠出现的次数。 可选参数 *start* 与 *end* 会被解读为切片表示法。

* `str.``find`(*sub*[, *start*[, *end*]])

  返回子字符串 *sub* 在 `s[start:end]` 切片内被找到的最小索引。 可选参数 *start* 与 *end* 会被解读为切片表示法。 如果 *sub* 未被找到则返回 `-1`。

* `str.``index`(*sub*[, *start*[, *end*]])

  类似于 find()，但在找不到子类时会引发 ValueError。

* 字典 `setdefault`(*key*[, *default*])

  如果字典存在键 key ，返回它的值。如果不存在，插入值为 default 的键 key ，并返回 default 。 default 默认为 None。

* 
