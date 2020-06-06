---
title: Python——回顾常用的库
date: 2019-05-05 18:42:08

tags:
  - Python
---

这些最基础的面试想不起来会很尴尬

<!--more-->

## Python map() 函数

### 描述

**map()** 会根据提供的函数对指定序列做映射。

第一个参数 function 以参数序列中的每一个元素调用 function 函数，返回包含每次 function 函数返回值的新列表。

语法

### map() 函数语法：

```python
map(function, iterable, ...)
```

### 参数

- function -- 函数
- iterable -- 一个或多个序列

### 返回值

Python 2.x 返回列表。

Python 3.x 返回迭代器。

### 实例

以下实例展示了 map() 的使用方法：

``````python
def square(x) :            # 计算平方数
	return x ** 2
>>>map(square, [1,2,3,4,5])   # 计算列表各个元素的平方
>>>[1, 4, 9, 16, 25]
>>>map(lambda x: x ** 2, [1, 2, 3, 4, 5])  # 使用 lambda 匿名函数
>>>[1, 4, 9, 16, 25]
``````



### 提供了两个列表，对相同位置的列表数据进行相加
``````python
>>>map(lambda x, y: x + y, [1, 3, 5, 7, 9], [2, 4, 6, 8, 10])
>>>[3, 7, 11, 15, 19]
``````





## Python filter() 函数

### 描述

**filter()** 函数用于过滤序列，过滤掉不符合条件的元素，返回由符合条件元素组成的新列表。

该接收两个参数，第一个为函数，第二个为序列，序列的每个元素作为参数传递给函数进行判，然后返回 True 或 False，最后将返回 True 的元素放到新列表中。

> **注意:** Pyhton2.7 返回列表，Python3.x 返回迭代器对象，具体内容可以查看：[Python3 filter() 函数](http://www.runoob.com/python3/python3-func-filter.html)

### 语法

以下是 filter() 方法的语法:

```
filter(function, iterable)
```

### 参数

- function -- 判断函数。
- iterable -- 可迭代对象。

### 返回值

返回列表。

------

### 实例

以下展示了使用 filter 函数的实例：

### 过滤出列表中的所有奇数：

``````python
#!/usr/bin/python 
# -*- coding: UTF-8 -*-   
def is_odd(n):
	return n % 2 == 1
newlist = filter(is_odd, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]) 
print(newlist)
``````



输出结果 ：

```python
[1, 3, 5, 7, 9]
```

### 过滤出1~100中平方根是整数的数：

``````python
#!/usr/bin/python 
# -*- coding: UTF-8 -*-   
import math 
def is_sqr(x):     
	return math.sqrt(x) % 1 == 0
newlist = filter(is_sqr, range(1, 101)) 
print(newlist)
``````



输出结果 ：

```python
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```





## Python reduce() 函数

### 描述

**reduce()** 函数会对参数序列中元素进行累积。

函数将一个数据集合（链表，元组等）中的所有数据进行下列操作：用传给 reduce 中的函数 function（有两个参数）先对集合中的第 1、2 个元素进行操作，得到的结果再与第三个数据用 function 函数运算，最后得到一个结果。

### 语法

reduce() 函数语法：

```
reduce(function, iterable[, initializer])
```

### 参数

- function -- 函数，有两个参数
- iterable -- 可迭代对象
- initializer -- 可选，初始参数

### 返回值

返回函数计算结果。

### 实例

以下实例展示了 reduce() 的使用方法：

``````python
>>>def add(x, y) :            # 两数相加 ...     
	return x + y ...  
>>> reduce(add, [1,2,3,4,5])   # 计算列表和：1+2+3+4+5 15 
>>> reduce(lambda x, y: x+y, [1,2,3,4,5])  # 使用 lambda 匿名函数 15
``````

