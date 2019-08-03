---
title: 13.找出数组中重复的数字
tags:
 - Python
 - 剑指offer
categories:
 - 算法
---
## 题目描述

给定一个长度为 nn 的整数数组 `nums`，数组中所有的数字都在 0∼n−10∼n−1 的范围内。

数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。

请找出数组中任意一个重复的数字。
<!-- more -->

**注意**：如果某些数字不在 0∼n−10∼n−1 的范围内，或数组中不包含重复数字，则返回 -1；

#### 样例

```
给定 nums = [2, 3, 5, 4, 3, 2, 6, 7]。

返回 2 或 3。
```

## 解决方案

思路：

创建一个字典用来保存已经出现过的数字，逐个比对即可。比较简单

``````python
class Solution(object):
    def duplicateInArray(self, nums):
        """
        :type nums: List[int]
        :rtype int
        """
        ser_dict = {}
        length = len(nums)
        res = -1
        for i in nums:
            if i <0 or i>=length:
                return -1
            if i in ser_dict:
                res = i
            else:
                ser_dict[i]=i
        return res
``````

时间复杂度： O(n)

空间复杂度：O(n)