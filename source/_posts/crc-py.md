---
title: python的CRC校验
date: 2019-10-15 15:13:07
tags:
 - Python
 - 网络
categories:
 - 计算机技术
---

## CRC校验

![KC8RKJ.png](https://s2.ax1x.com/2019/10/15/KC8RKJ.png)
<!-- more -->

> 循环冗余校验码（CRC），简称循环码，是一种常用的、具有检错、纠错能力的校验码，在早期的通信中运用广泛。循环冗余校验码常用于外存储器和计算机同步通信的数据校验。奇偶校验码和海明校验码都是采用奇偶检测为手段检错和纠错的(奇偶校验码不具有纠错能力)，而循环冗余校验则是通过某种数学运算来建立数据位和校验位的约定关系的。
>
> ——百度百科

## crcmod校验CRC

`crcmod`该软件包中的软件是一个Python模块，用于生成计算循环冗余校验（`CRC`）的对象。它包括一个用于快速计算的（可选）C扩展，以及一个纯Python实现。

该程序包允许使用任何8、16、24、32或64位`CRC`。您可以为选定的多项式或[`crcmod.Crc`](http://crcmod.sourceforge.net/crcmod.html#crcmod.Crc)类的实例生成Python函数，该实例 提供与Python标准库中的[`hashlib`](http://docs.python.org/library/hashlib.html#module-hashlib)，[`md5`](http://docs.python.org/library/md5.html#module-md5)和[`sha`](http://docs.python.org/library/sha.html#module-sha)模块相同的接口 。

### 安装

```
pip install crcmod
```

### 算法参数

| Name           | Polynomial | Reversed | Init-value | XOR-out | Check  |
| :------------- | :--------- | :------- | :--------- | :------ | :----- |
| `crc-8`        | 0x107      | False    | 0x00       | 0x00    | 0xF4   |
| `crc-8-darc`   | 0x139      | True     | 0x00       | 0x00    | 0x15   |
| `crc-8-i-code` | 0x11D      | False    | 0xFD       | 0x00    | 0x7E   |
| `crc-8-itu`    | 0x107      | False    | 0x55       | 0x55    | 0xA1   |
| `crc-8-maxim`  | 0x131      | True     | 0x00       | 0x00    | 0xA1   |
| `crc-8-rohc`   | 0x107      | True     | 0xFF       | 0x00    | 0xD0   |
| `crc-8-wcdma`  | 0x19B      | True     | 0x00       | 0x00    | 0x25   |
| `crc-16`       | 0x18005    | True     | 0x0000     | 0x0000  | 0xBB3D |

### CRC16校验

`crcmod.mkCrcFun`的参数解析

- *poly* –用于计算CRC的生成多项式。该值指定为Python整数或长整数。该整数中的位是多项式的系数。允许的唯一多项式是那些生成8、16、24、32或64位CRC的多项式。
- *initCrc* –用于开始CRC计算的初始值。该初始值应为初始移位寄存器值，如果使用反向算法则应反向，然后与最终XOR值进行XOR。这等效于算法应针对零长度字符串返回的CRC结果。默认设置为所有位，因为该起始值将考虑前导零字节。从零开始将忽略所有前导零字节。
- *rev-*当为`True`时，选择位反转算法的标志。默认为 `True，`因为位反转算法更有效。
- *xorOut* –与计算出的CRC值进行XOR的最终值。由某些CRC算法使用。默认为零。

代码实现

```python
import crcmod


words = b'Y\x11\x9b\xdf\x00\x00\n\x88$\x10\x00\x00\x00\x01\xa0\xcc\xb5'
crc16 = crcmod.mkCrcFun(0x18005, rev=True, initCrc=0xFFFF, xorOut=0x0000)
print(crc16(words))

# 42443

```

## 参考资料：

> http://crcmod.sourceforge.net/crcmod.html
>
> https://stackoverflow.com/questions/35205702/calculating-crc16-in-python
>
> [https://baike.baidu.com/item/%E5%BE%AA%E7%8E%AF%E5%86%97%E4%BD%99%E6%A0%A1%E9%AA%8C%E7%A0%81/10168758?fromtitle=CRC%E6%A0%A1%E9%AA%8C&fromid=3439037](https://baike.baidu.com/item/循环冗余校验码/10168758?fromtitle=CRC校验&fromid=3439037)

