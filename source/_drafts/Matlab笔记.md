---
title: Matlab笔记
tags:
---

# Matlab快速入门
## 桌面基础知识
1. 变量创建
   * 类似Python
   * 加分号就不会显示答案，而是单纯执行
     ```matlab
    a = b + c
      % 输出a = b + c的值
      a = b + c
      % 无输出
     ```

2. 内置了一些函数可以直接用，如`sin`,`cos`等

## 矩阵和数组

### 数组创建

1. 直接创建

   ```matlab
a = [1,2,3]
	% 单行数组，实际上为行向量
	a = [1,2,3; 4,5,6; 7,8,9]
	% 多行矩阵
	```
	
2. 使用内置函数，如`ones`，`zeros`,`rand`

   ```matlab
   z = zeros(5,1)
   ```

### 矩阵和数组运算

1. 类似于`numpy`中的广播机制，矩阵和数字的运算（或者将函数应用于矩阵）会被广播到矩阵的每一个元素

2. 矩阵转置

   `a'`

3. 矩阵的逆

   `inv(a)`

4. 矩阵乘法

   `p = a * b`

5. 矩阵元素乘法（每一个元素相乘）

   `p = a.*a` 使用`.*`，同样的也有`a.^3`表示对`a`中的每一个元素算三次方，`a./3`表示对每一个元素计算除以3的结果

6. 内容的显示位数

   `format [format]`其中`format`可为`long`,`short`等等

7. 串联

   通过`[]`来串联不同的矩阵，提供`行串联`以及`列串联`

   ```matlab
   A = [a, a]
   % 行串联
   A = [a; a]
   % 列串联
   ```

8. 复数，可以直接使用`i`或者`j`表示复数的虚部

## 数组索引

类似于`numpy`的数组索引

越界的`赋值`访问会将矩阵进行扩充来容纳新的数据，而不会报错

`A([start:step:end],[start:step:end])`

## 工作区变量

1. 使用`whos`来查看工作区内容，如下图

   ![](/home/samuel/git-repos/blogs/source/_drafts/Matlab笔记/whos.png)

   可以加上具体的变量名来查看该变量的信息

   `whos variable`

2. 通过`save`命令将工作区的变量保存为`.mat`文件

   `save myfile.mat`

   并通过`load`将`.mat`文件中的数据还原到工作区

   清空工作区变量`clear`

## 文本和字符

### 字符串数组中的文本

```matlab
t = "Hello, world";
q = "Something ""quoted"" and something else"
% 文本本身包含双引号需要使用两个双引号
% q,t均为数组,数组类型为string,形状为1x1
```

1. 文本拼接使用`+`号

2. `strlength`函数用于获取字符串长度

### 字符数组中的数据

```matlab
seq = 'GCTAGAATCC';
% seq为char类型的数组,而不是一个整体
% 所以拼接的时候类似于串联数值数组
seq2 = [seq 'ATTAGAAACC']
% 加不加逗号进行分割都可以,个人还是觉得加要好一些
```

接收`string`数据的所有函数都可以接收`char`数据,反之亦然

## 调用函数



1. 如果一个函数有多个输出结果,需要将接收者放到方括号中

   > 与Python不同，Python不需要加`[]`

   ```matlab
   A = [1 3 5];
   [maxA, location] = max(A);
   ```

   `clc`函数清空命令窗口，如同`clear`

## 二维图和三维图

### 线图

使用`plot`函数绘制二维线图

```matlab
x = 0:pi/100:2*pi;
y = sin(x);
plot(x,y)
```

标记轴并且添加标题

```matlab
xlabel('x')
ylabel('sin(x)')
title('Plot of the Sine Function')
```

第三个参数，可以控制绘制的颜色以及形状

```matlab
plot(x,y,'r--')
```

> 'r--' 为线条设定。每个设定可包含表示线条颜色、样式和标记的字符。标记是在绘制的每个数据点上显示的符号，例如，+、o 或 *。例如，'g:*' 请求绘制使用 * 标记的绿色点线。

将多个图画到同一个窗口

```matlab
x = 0:pi/100:2*pi;
y = sin(x);
plot(x,y)

hold on

y2 = cos(x);
plot(x,y2,':')
legend('sin','cos')

hold off
```

### 三维绘图

使用`surf`函数

> `surf` 函数及其伴随函数 `mesh` 以三维形式显示曲面图。`surf` 使用颜色显示曲面图的连接线和面。`mesh` 生成仅以颜色标记连接定义点的线条的线框曲面图。

### 子图

使用`subplot`

```matlab
t = 0:pi/10:2*pi;
[X,Y,Z] = cylinder(4*cos(t));
subplot(2,2,1); mesh(X); title('X');
subplot(2,2,2); mesh(Y); title('Y');
subplot(2,2,3); mesh(Z); title('Z');
subplot(2,2,4); mesh(X,Y,Z); title('X,Y,Z');
```

说不清楚，用到的时候再说吧

## 编程和脚本

### 脚本

创建脚本，使用`edit`命令

`edit mysphere`

使用`%`来注释

### 实时脚本

创建文件的时候加上`.mlx`拓展名

`edit newfile.mlx`

### 循环以及条件语句

#### 循环

```matlab
N = 100
f(1) = 1;
f(2) = 1;
for n = 3:N
	f(n) = f(n-1) + f(n-2);
end
f(1:10)
```

#### 条件

```matlab
num = randi(100)
if num < 34
   sz = 'low'
elseif num < 67
   sz = 'medium'
else
   sz = 'high'
end
```

## 帮助和文档

1. `doc`

   `doc mean`在单独的窗口中打开函数`mean`的文档

2. 在键入函数输入参数的左括号之后暂停，此时命令行窗口中会显示相应函数的提示（函数文档的语法部分）。

3. 使用 help 命令可在命令行窗口中查看相应函数的简明文档。

   `help mean`

## 矩阵和幻方矩阵

### 生成矩阵

1. `zeros`
2. `ones`
3. `rand`
4. `randn`

> 其他的一些点，比如矩阵和，没什么好说的，过于细节，用到的时候再看也行

## 表达式

### 变量

最长变量名 = `namelengthmax`

### 函数

`help elfun`访问初等数学函数列表

`help specfun`

`help elmat` 访问高等数学函数以及矩阵函数的列表

### 可能用到的一些常量

1. `pi`
2. `i` / `j`
3. `Inf`
4. `NaN`

## 输入命令

### `format`函数

> 只影响数字显示的方式，而不影响数字计算的精度

### 取消输出

使用分号结尾，在生成大型矩阵的时候很有用

### 输入长语句

使用`...`在语句末尾以指示该语句在下一行继续

## 索引

### 下标

### 冒号运算符

`1:10`是包含`1`到`10`之间整数的行向量

### 串联

### 删除行和列

对要删除的元素(列、行、元素)赋值为`[]`

```matlab
X = A
X(:, 2) = []
% 删除第二列
X(2:2:10) = []
% 删除指定元素（集合），将剩余元素重构为一个行向量
```

### 逻辑下标

类似于`numpy`和`pandas`，可以认为是先生成了一个类型为`logical`的矩阵，然后对原始矩阵(数组)进行索引，表示是否选取

> 用`isfinite`来排除`NaN`以及`Inf`

### `find`函数

## 数组类型

### 多维数组

### 元胞数组

1. 以其他数组为副本，也就是改变元胞不会反映到原数组
2. 可以包含不同大小的矩阵序列
3. 使用`{}`来进行索引

### 结构体

1. 支持动态字段名称
2. 支持单个结构体拓展为数组?(不知道如何描述)建议看[官方文档](https://ww2.mathworks.cn/help/matlab/learn_matlab/types-of-arrays.html)

## 线性代数

1. 单位矩阵使用`eye`函数来生成，如`eye(n)`生成`n×n`的单位矩阵
2. 矩阵的逆`inv(A)`，使用`det`计算矩阵线性变换的缩放因子

> 有些矩阵接近***奇异矩阵***，但是计算容易出现数值误差，`cond`函数计算逆矩阵的条件数，它指示矩阵求逆结果的精度，条件数的范围从`1`（数值稳定的矩阵），到`Inf`（奇异矩阵）

3. `Kronecker`张量积，没听说过
4. 范数`norm函数`

## 非线性函数的运算

