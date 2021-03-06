---
title: 每日一题:leetcode:100. 相同的树
date: 2020-04-20T09:02:46+08:00
tags:
 - Python
 - leetcode
---
### 问题描述

[每日一题:leetcode:100. 相同的树](https://leetcode-cn.com/problems/same-tree/)

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。
<!-- more -->
**示例 1:**

```
输入:       1         1
          / \       / \
         2   3     2   3

        [1,2,3],   [1,2,3]

输出: true
```

**示例 2:**

```
输入:      1          1
          /           \
         2             2

        [1,2],     [1,null,2]

输出: false
```

**示例 3:**

```
输入:       1         1
          / \       / \
         2   1     1   2

        [1,2,1],   [1,1,2]

输出: false
```

### 题解

```
> 类型：DFSF分制
> Time Complexity O(N)
> Space Complexity O(h)
```

在每一层先检查再递归，所以这是pre-order的思路。
比对相等的条件：

1. `p.val == q.val`
2. `if not p or not q: return p == q`
   如有不等，直接返回False，就不用继续递归了。最后左右孩子返回给Root：`return left and right`

p.s. 上面的第二个相等条件，检查了2种情况：
1.`if not p and not q: return True`
2.`if not p or not q: return False`



1. 递归

``````python
class Solution(object):
    def isSameTree(self, p, q):
        if not p or not q: return p == q
        if p.val != q.val: return False
        
        left = self.isSameTree(p.left, q.left)
        right = self.isSameTree(p.right, q.right)
        return left and right
``````

2. stack(DFS iteratively)

``````python
# DFS iteratively
class Solution2:
    def isSameTree(self, p: TreeNode, q: TreeNode) -> bool:
        stack = [(p, q)]
        while stack:
            p, q = stack.pop()
            if not p and not q:
                continue
            elif (not q or not p) or (p.val != q.val):
                return False
            stack.extend([(q.right, p.right), (q.left, p.left)])
        return True
``````

3. queue(BFS iteratively)

``````python
# BFS iteratively
class Solution3:
    def isSameTree(self, p: TreeNode, q: TreeNode) -> bool:
        queue = collections.deque([p, q])
        while queue:
            p, q = queue.popleft()
            if not p and not q:
                continue
            elif (not p or not q) or (p.val != q.val):
                return False
            queue.extend([(p.left, q.left), (p.right, q.right)])
        return True
``````
