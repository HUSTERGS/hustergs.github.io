---
title: python-cookbook-ch3
categories:
  - Programming Language
  - Python
abbrlink: fb96166c
date: 2020-06-03 16:34:44
tags:
---


## Chapter 3
> Numbers, Dates, and Times

1. 舍入小数到某一指定小数点
   ```python
   round(1.23, 1) # 1.2
   round(1.27, 1) # 1.3
   ```
   采用四舍五入,如果第二个参数为负数,那么就会指示十位百位,,,以此类错
   ```python
   round(1627731, -1) # 1627730
   ```

2. 进行**准确**的十进制计算
   ```python
   # Python, js等语言均有类似的问题
   a = 4.2
   b = 2.1
   a + b # 6.300000000000001
   (a + b) == 6.3 # False
   ```
   想要避免这种问题,使用`Decimal`
   ```python
   from decimal import Decimal
   a = Decimal('4.2')
   b = Decimal('2.1')
   a + b # Deciaml('6.3')
   (a + b) == Decimal('6.3') #  True
   (a + b) == 6.3 #  False
   ```
<!-- more -->
3. Python中的8进制与其他的语言不太相同
   二进制以`0b`开头
   十六进制以`0x`开头
   **八进制以`0o`开头**
   所以
   ```python
   os.chmod('script.py', 0755) # error
   ## 改为以下
   os.chmod('script.py', 0o755) # error
   ```

4. 不仅仅可以从`math`中引入`inf`，也可以直接使用`float('inf')`来创建，同样的还有
   - `float('nan')`
   - `float('-inf')`
   但是需要注意的是，`nan`之间无法进行相等判断(`inf`可以)，如
   ```python
   a = float('nan')
   b = float('nan')
   a == b # False
   a is b # False
   ```
   所以如果要测试某一个数是否是`nan`，最好的办法就是使用`math.isnan`，虽然`inf`本身可以比较，但是也可以使用`math.isinf`来判断

5. 使用`Fraction`来进行分数计算
   ```python
   from fractions import Fraction
   a = Fraction(5, 4)
   b = Fraction(7, 16)
   a + b # 27/16

   # 转化为小数
   float(a)
   # 获得分子与分母
   ## 分子
   a.numerator # 5
   a.denominator # 4

   # 创建时，如果可以约分将自动约分，且通过`numerator`/`denominator`获得的分子分母也为约分之后的分子分母
   c = Fraction(4, 18) # 等同于 c = Fraction(2, 9)
   # 可以将一个小数直接转化为分数
   d = 3.75
   y = Fraction(*x.as_integer_ratio()) # Fraction(15, 4)
   ## 但是同样有可能遇到问题
   Fraction(*1.2.as_integer_ratio()) # Fraction(5404319552844595, 4503599627370496)
   ## 可以通过上述的Decimal来获得正确的结果
   d = Decimal('1.2')
   Fraction(*d.as_integer_ration()) # (6,5)
   ```

6. Python中的list可以通过乘法来扩展
   ```python
   a = [1]
   a * 10 # [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
   ```
   同样可以扩展到字符串
   ```python
   a = '123'
   a * 2 # '123123'
   ```
   有时候会很有用，尤其是字符串
  
7. 日期相关
   用到再看，虽然csp里面好几次有相关的题目，奈何确实太繁琐
  