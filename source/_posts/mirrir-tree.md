---
title: 19. 二叉树的镜像
date: 2019-08-14 19:31:59
tags:
 - Python
---

## 问题描述

输入一个二叉树，将它变换为它的镜像。

<!-- more -->

#### 样例

```python
输入树：
      8
     / \
    6  10
   / \ / \
  5  7 9 11

 [8,6,10,5,7,9,11,null,null,null,null,null,null,null,null] 
输出树：
      8
     / \
    10  6
   / \ / \
  11 9 7  5

 [8,10,6,11,9,7,5,null,null,null,null,null,null,null,null]
```

## 解决方案

``````python
# -*- coding:utf-8 -*-
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    # 返回镜像树的根节点
    def Mirror(self, root):
        # write code here
        if not root:
            return root
        root.left,root.right = root.right,root.left
        self.Mirror(root.left)
        self.Mirror(root.right)
        return root
``````

