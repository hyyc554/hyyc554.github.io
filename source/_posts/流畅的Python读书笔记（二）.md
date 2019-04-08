---
title: 流畅的Python读书笔记（二）
tags:
 - Python
---

![](https://ws1.sinaimg.cn/large/d126accegy1fyx4jkxhnaj22l51xu4j6.jpg)

<!--more-->

## 2.1 可变序列与不可变序列

- 可变序列
  - list、 bytearray、 array.array、 collections.deque 和 memoryview。 

- 不可变序列
  - tuple、 str 和 bytes。 

![](https://ws1.sinaimg.cn/large/d126accegy1fyvzw4nki0j20h805gmyj.jpg)

## 2.2　列表推导和生成器表达式 

- 列表推导是构建列表（list）的快捷方式
- 生成器表达式则可以用来创建其他任何类型的序列 

ps:

**很多 Python 程序员都把列表推导（list comprehension）简称为 listcomps，生成*
式表达器（generator expression）则称为 genexps。* 

### 2.2.1　列表推导和可读性 

新手写法：

``````python
>>> symbols = '$¢£¥€¤'
>>> codes = []
>>> for symbol in symbols:
... 	codes.append(ord(symbol))
...
>>> codes
[36, 162, 163, 165, 8364, 164]
``````

列表推导写法：

``````python
>>> symbols = '$¢£¥€¤'
>>> codes = [ord(symbol) for symbol in symbols]
>>> codes
[36, 162, 163, 165, 8364, 164]
``````

列表推导不会再有变量泄漏的问题 

### 2.2.2　列表推导同filter和map的比较 

实例代码：

``````python
>>> symbols = '$¢£¥€¤'
>>> beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]
>>> beyond_ascii
[162, 163, 165, 8364, 164]


>>> beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))
>>> beyond_ascii
[162, 163, 165, 8364, 164]
``````

下面的这种写法，确实很难看。。。

### 2.2.3　笛卡儿积

笛卡尔积定义：

> 笛卡尔乘积是指在数学中，两个[集合](https://baike.baidu.com/item/%E9%9B%86%E5%90%88)*X*和*Y*的笛卡尓积（Cartesian product），又称[直积](https://baike.baidu.com/item/%E7%9B%B4%E7%A7%AF)，表示为*X*×*Y*，第一个对象是*X*的成员而第二个对象是*Y*的所有可能[有序对](https://baike.baidu.com/item/%E6%9C%89%E5%BA%8F%E5%AF%B9)的其中一个成员[3]  。
>
> 假设集合A={a, b}，集合B={0, 1, 2}，则两个集合的笛卡尔积为{(a, 0), (a, 1), (a, 2), (b, 0), (b, 1), (b, 2)}。

如果你需要一个列表，列表里是 3 种不同尺寸的 T 恤衫，每个尺寸都有 2 个颜色，示例
2-4 用列表推导算出了这个列表，列表里有 6 种组合。 

``````python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> tshirts = [(color, size) for color in colors for size in sizes]
>>> tshirts
[('black', 'S'), ('black', 'M'), ('black', 'L'), ('white', 'S'),
('white', 'M'), ('white', 'L')]
>>> for color in colors: 
... 	for size in sizes:
... 		print((color, size))
...
('black', 'S')
('black', 'M')
('black', 'L')
('white', 'S')
('white', 'M')
('white', 'L')
>>> tshirts = [(color, size) for size in sizes 
               				 for color in colors]
>>> tshirts
[('black', 'S'), ('white', 'S'), ('black', 'M'), ('white', 'M'),
('black', 'L'), ('white', 'L')]
``````

通过调整for的顺序，来实现对内容的排序方式

### 2.2.4　生成器表达式 

虽然也可以用列表推导来初始化元组、数组或其他序列类型，但是**生成器表达式是更好的**
**选择**。

这是因为生成器表达式背后遵守了迭代器协议，可以逐个地产出元素，而不是先建立一个完整的列表，然后再把这个列表传递到某个构造函数里。前面那种方式显然能够**节省内存**。 

生成器表达式的语法跟列表推导差不多，只不过把**方括号换成圆括号**而已。 

``````python
>>> symbols = '$¢£¥€¤'
>>> tuple(ord(symbol) for symbol in symbols)
(36, 162, 163, 165, 8364, 164)
>>> import array
>>> array.array('I', (ord(symbol) for symbol in symbols))
array('I', [36, 162, 163, 165, 8364, 164])
``````



``````python
>>> colors = ['black', 'white']
>>> sizes = ['S', 'M', 'L']
>>> for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes): ➊
... 	print(tshirt)
...
black S
black M
black L
white S
white M
white L
``````

## 2.3　元组不仅仅是不可变的列表 

元祖可以用于没有字段名的记录 

### 2.3.1　元组和记录 

元组其实是对数据的记录：元组中的每个元素都存放了记录中一个字段的数据，外加这个字段的位置。正是这个位置信息给数据赋予了意义 。

如果把元组当作一些字段的集合，那么数量和位置信息就变得非常重要了 

如果在任何的表达式里我们在元组内对元素排序，这些元素所携带的信息就会丢失，因为这些信息是跟它们的位置有关的 

``````python
>>> lax_coordinates = (33.9425, -118.408056) ➊
>>> city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014) ➋
>>> traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ➌
... ('ESP', 'XDA205856')]
>>> for passport in sorted(traveler_ids): ➍
... print('%s/%s' % passport) ➎
...
BRA/CE342567
ESP/XDA205856
USA/31195855
>>> for country, _ in traveler_ids: ➏
... print(country)
...
USA
BRA
ESP
``````

for 循环可以分别提取元组里的元素，也叫作拆包（unpacking）。因为元组中第二个元素对我们没有什么用，所以它赋值给“_”占位符。 

拆包让元组可以完美地被当作记录来使用 

总结：记录与元祖——位置的重要性

### 2.3.2　元组拆包 

`*`运算符把一个可迭代对象拆开作为函数的参数：

``````python
>>> divmod(20, 8)
(2, 4)
>>> t = (20, 8)
>>> divmod(*t)
(2, 4)
>>> quotient, remainder = divmod(*t)
>>> quotient, remainder
(2, 4)
``````

**用*来处理剩下的元素** 

不仅仅可以用在*args

``````
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(3)
>>> a, b, rest   
(0, 1, [2])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])
``````

可以出现在任意位置

``````python
>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
>>> *head, b, c, d = range(5)
>>> head, b, c, d
([0, 1], 2, 3, 4)
``````

### 2.3.3　嵌套元组拆包 

 ``````python
metro_areas = [
('Tokyo','JP',36.933,(35.689722,139.691667)), # ➊
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.'))
fmt = '{:15} | {:9.4f} | {:9.4f}'
for name, cc, pop, (latitude, longitude) in metro_areas: # ➋
	if longitude <= 0: # ➌
		print(fmt.format(name, latitude, longitude))
 ``````

