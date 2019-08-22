---
title: 剪绳子
date: 2019-08-21 19:46:57
tags:
 - Python
 - 剑指offer
categories:
 - 算法
---



## 问题描述

给你一根长度为 nn 绳子，请把绳子剪成 mm 段（mm、nn 都是整数，2≤n≤582≤n≤58 并且 m≥2m≥2）。

每段的绳子的长度记为k[0]、k[1]、……、k[m]。k[0]k[1] … k[m] 可能的最大乘积是多少？

例如当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到最大的乘积18。

<!--more-->

## 解决方案

``````python
class Solution(object):
    def maxProductAfterCutting(self,length):
        """
        题目解析：
        用来找一个剪绳子的方案，使得减绳子之后的每段乘积大于当前绳子总长度
        解法：n表示绳子总长度，m表示剪绳子的段数
        1.如果绳子总长度<4，那么减绳子之后的乘积小于绳子总长度
        2.如果绳子总长度=4，那么可以将绳子减为2段，此时每段乘积和总长度相等
        3.如果绳子总长度>=5，那么剪绳子的乘积肯定存在某个值大于绳子总长:
            可以证明2(n-2)>n并且3(n-3)>n。
            而且3(n-3)>=2(n-2)。
            所以我们应该尽可能地多剪长度为3的绳子段，长度为2的绳子最多2段，不要留绳子长度为1的

        :type length: int
        :rtype: int
        """
        # 边界判断
        if length<2: return 0
        elif length==2: return 1
        elif length==3: return 2
        else:
            #其他情况，如果总的绳子长度=4，那么效果一样，最大乘积都是4
            #如果绳子长度>=5,那么需要尽可能的多剪成长度为3的子绳子段,然后长度为2的绳子最多2段，不要留绳子长度为1的
            times_of_three = length //3
            spare = length % 3
            if spare==1:
                times_of_three-=1
            times_of_two = spare//2
            res = pow(3,times_of_three)*pow(2,times_of_two)
            return res

def test_Solution():
    a = Solution()
    assert a.maxProductAfterCutting(8) == 18
    # assert a.maxProductAfterCutting(10) == 18
``````

