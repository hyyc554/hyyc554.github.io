---
title: 斐波那契数列
date: 2019-09-03 19:41:37
tags:
 - 算法
 - 剑指offer
categories:
 - 计算机技术

---

## 问题描述

输入一个整数 nn ，求斐波那契数列的第 nn 项。

假定从0开始，第0项为0。(nn<=39)

#### 样例

```
输入整数 n=5 

返回 5
```

## 解决方案

``````python
class Solution(object):
    def Fibonacci(self, n):
        """
        :type n: int
        :rtype: int
        """

        tempArray = [0, 1]
        if n >= 2:
            for i in range(2, n+1):
                tempArray[i%2] = tempArray[0] + tempArray[1]
        return tempArray[n%2]
``````

