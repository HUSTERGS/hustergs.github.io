---
title: python-cookbook-ch1
tags: Python
categories:
  - Programming Language
  - Python
abbrlink: '15987740'
date: 2020-05-27 19:19:05
---

## 前言
> 本书介绍了Python应用在各个领域中的一些使用技巧和方法，从最基本的字符、文件序列、字典和排序，到进阶的面向对象编程、数据库和数据持久化、 XML处理和Web编程，再到比较高级和抽象的描述符、装饰器、元类、迭代器和生成器，均有涉及。书中还介绍了一些第三方包和库的使用，包括 Twisted、GIL、PyWin32等。本书覆盖了Python应用中的很多常见问题，并提出了通用的解决方案。书中的代码和方法具有很强的实用性，可以方便地应用到实际的项目中，并产生立竿见影的效果。尤为难得的是，本书的各位作者都具有丰富的业界实践经验，因此，本书不仅给出了对各种问题的解决方案，同时还体现了很多专家的思维方式和良好的编程习惯，与具体的细节性知识相比，这部分内容无疑是本书的精华。

很久之前看过一部分，基本上都忘记了，现在又重拾这本书，打算把平时经常用到的几章先看完(其实就是不想做课内的各种实验:()，主要记录一些我不知道，或者我知道，但是依然觉得比较重要的点(指csp)。一些本人平时已经用的很多，或者觉得未来用到的几率很小的点将不会记录。由于部分使用了书中的示例代码，如有侵权，请联系删除


## Chapter 1
> Data Structures and Algorithms


1. 解包操作可以嵌套,使用`_`来丢弃不需要的值
   ```python
   data = ['ACME', 50, 91.1, (2012, 12,21)]
   name, shares, price, (year, mon, day) = data
   _, shares, price, _ = data
   ```
2. 不定长度的解包,通过`*`表达式来解决
   ```python
   record = ('Dave', 'dave@example.com', '123', '456')
   name, email, *phone_numbers = user_record
   ```
   使用单独的`*`或者`*_`来丢弃不需要的值
   ```
   record =  ['ACME', 50, 91.1, (2012, 12, 21)]
   name, *+, (year, *_) = record
   ```
3. 使用`deque(maxlen=N)`来创建一个只有固定长度的队列，当加入新的成员而队列已满的时候将会自动将最早加入的成员推出。虽然也可以自己通过一些逻辑来实现，但是这种方法不但简洁而且更快

<!-- more -->

5. 获得`N`个元素中最大/最小的`k`个元素
   ```python
   import heapq
   nums = [1,2,3,4]
   heapq.nlargest(3, nums)
   heapq.nsmallest(3,nums)
   ```
   可以通过`key`参数来决定被排序的具体表达式
   > 但是似乎无法方便的自定义比较函数，需要将比较对象包装在一个类中，并重写`__lt__()`方法
   两个函数的底层是通过先`heapify`再`pop`来依次获取`k`个最小/最大的元素，而`heapify`的复杂度是$O(log N)$，所以整体的复杂度是$O(klog N)$，但是
   如果直接排序则复杂度是$O(Nlog N)$，这种方法在`N` >> `k`时尤为有效
6. 使用`defaultdict`来自动在访问不存在的键的时候创建对应的值
   ```python
   from collections import defaultdict
   d = defaultdict(list) # 参数为值的类型
   d['a'].append(1)
   a['a'].append(2) # 直接访问并加入，不需要预先判断
   ```
   正常来说，如果不使用`defaultdict`则代码如下
   ```python
   d = {}
   for key, value in pairs:
      if key not in d:
         d[key] = []
      d[key].append(value)
   ```
   如果使用`defaultdict`则整个代码逻辑大大简化
   ```python
   d = defaultdict(list)
   for key, value in pairs:
      d[key].append(value)
   ```
   需要注意的点就是，在访问的时候就会创建，有时可能会引起一些不必要的麻烦

7. 有序字典
   使用`OrderedDict`使得我们可以根据插入的顺序来遍历一个字典，重新赋值不会改变这个顺序
   ```python
   from collections import OrderedDict
   d = OrderedDict()
   d['foo'] = 1
   d['bar'] = 2
   for key in d:
      print(key, d[key]) # 顺序打印foo 以及bar
   ```
   该结构的内部使用了双向链表来记录顺序，所以相比一般的字典所占用的空间大了两倍不止，所以使用的时候需要进行权衡
   > 感觉如果真的有这样的需求，并且自己来实现的话，其实也需要维护一个list，说不准哪一种会更好，但是对于那种遍历的顺序就是简单的range(n)的话，应该还是自己实现的更好一些

8. 尝试使用`set()`来判断计算交集并集等值
   ```python
   a = {
      'x': 1,
      'y': 2,
      'z': 3
   }
   b = {
      'w': 4,
      'x': 11,
      'y': 2
   }
   a.keys() & b.keys() # {'x', 'y'}
   a.items() & b.items() # {('y', 2)}
   ```
9. 使用`slice`使得切片操作不再是硬编码的，而是通过变量名能够表达更多的含义，如
   ```python
   items = [0,1,2,3,4]
   a = slice(2, 4)
   items[2:4] # [2,3]
   items[a] # [2,3]
   ```
   > 其实感觉这个用到的很少，常常且切片的时候会用到被切片者的一些属性，如len等等，当然这种情况下也可以使用`slice`

10. 计算一个序列中出现次数最多的元素以及其出现的个数,一般来说我用的是`np.unique`函数，但是有些情况下不允许使用第三方的包，那么Python也提供了内置的解决方案，即`collections.Counter`
    ```python
    from collections import Counter
    word_conuts = Conuter(words)
    top_three = word_counts.most_common(3) # 返回一个二元组的list，二元组的格式为(value, count)
    ```
    而Counter不仅仅提供了`most_common`这种功能，也有一些`np.unique`所不具备的功能(也有可能是我不知道:-))，即两个Counter对象的数学运算，如加法
    ```python
    a = Counter(words)
    b = Counter(morewords)
    c = a + b # c 中会存储两个统计表中每个元素出现次数的和
    ```

11. 使用`itemgetter`而不是`lambda`表达式来提高效率，用于对字典list排序
    ```python
    rows = [
       {'a': 1, 'b':2},
       {'a': 2, 'b':3},
       {'a': 1, 'b':4},
       {'a': 4, 'b':2}
    ]
    # 简单的写法
    sorted(rows, key=lambda r:r['a'])
    sorted(rows, key=lambda r:r['b'])

    # 使用`itemgetter`
    from operator import itemgetter
    sorted(rows, key=itemgetter('a'))
    ```
    > 本来不准备记录这一条的，且不说这种需求常不常见，在后续的Python版本中，可能连性能优势都会变得不那么明显
   同样的也有`attrgetter`用于一般的对象之间比较时使用某一个属性所谓比较对象，给出的理由同样是有*一定的*性能优势
11. 使用`()`而不是`[]`以生成一个一个迭代器而不是列表，从而降低空间消耗
