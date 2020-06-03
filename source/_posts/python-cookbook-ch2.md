---
title: python-cookbook-ch2
tags: Python
categories:
  - Programming Language
  - Python
abbrlink: 8c9126fa
date: 2020-06-03 10:26:22
---


## Chapter 2
> Strings and Text

1. 在正则表达式中使用括号来捕捉目标内容
   ```python
   datepat = re.complie(r'(\d+)/(\d+)/(\d+)')
   m = datepat.match('11/27/2012')
   m.group(0) # 11/27/2012
   m.group(1) # 11
   m.group(2) # 27
   m.group(3) # 2012
   m.groups() # ('11', '27', '2012')
   ```
   > 注意在正则表达式中常常需要使用`raw`字符串,即在字符串前加上`r`,表示其中的内容不应该进行转义,否则上述的正则表达式必须写为`'(\\d+)/(\\d+)/(\\d+)'`这个地方可能会产生如下疑问*如果不进行转义的话,那怎么识别`\n`这种换行符号呢*,可以这样理解,regex和python原生的字符串使用的是两套系统,在使用raw字符串的时候,可以认为python的系统完全被屏蔽,字符串中的内容会被传入regex系统,并将其中的字符进行regex系统的转义,如`\d`代表一个数字,`\s`代表一个连续字符串,等等.其中`\n`不仅在Python系统中是转义字符,在regex系统中也是转义字符,所以使用raw regex的时候,字符串中的`\n`确实会被理解为两个字符,而不是换行符,但是在传入regex进行处理的时候,会被再一次转义,处理为换行符. 通过实验也可以发现
   ```python
   re.match('\n', '\n')
   re.match('\\n', '\n')
   re.match(r'\n', '\n')
   ```
   > 均可以正常匹配
   参见[What exactly is a “raw string regex” and how can you use it?](https://stackoverflow.com/questions/12871066/what-exactly-is-a-raw-string-regex-and-how-can-you-use-it)

<!-- more -->

2. 使用`re.sub`进行字符串的替换,重点在于可以在正则表达式中使用`()`来捕获目标并通过`\number`的形式在替换的字符串中使用
   ```python
   text = 'Today is 11/27/2012, PyCon starts 3/13/2013.'
   re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
   ## 'Today is 2012-11-27, PyCon starts 2013-3-13.'
   ```
   其中`number`可以理解为传入`group`函数的参数,不同的是只能从1开始

3. `strip`以及相关的`lstrip`,`rstrip`可以接收参数来指定去除的对象
4. 使用
   - `ljust()`
   - `rjust()`
   - `center()`
   - `format`
   函数来进行文本的对齐`Aligning Text Strings`
   ```python
   text = 'Hello World'
   text.ljust(20) # 'Hello World         '
   text.rjust(20) # '         Hello World'
   text.center(20)# '    Hello World     '
   # 同时可以接收参数，设置填充的字符
   text.center(20)# '****Hello World*****'
   # 在format函数中使用`>`, `<`以及`^`符号来达到相同的效果
   format(text, '>20') # '         Hello World'
   format(text, '<20') # 'Hello World         '
   ```
5. 活用`f'{variable}'`以及`'{variable}'.format(variable=name)`

   