---
title: LeetCode（[21. 合并两个有序链表]）
date: 2019-05-05 18:42:08

tags:
 - leetcode
---

## 问题描述

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 \

<!--more-->

**示例：**

```
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```

## 解决方案

``````python
# encoding: utf-8

class Node(object):
    def __init__(self):
        self.val = None
        self.next = None

    def __str__(self):
        return str(self.val)


def mergeTwoLists(l1, l2):
    # 处理边界情况（l1或l2为空）
    if l1 is None:
        return l2
    if l2 is None:
        return l1
    # 确保l1有最小的初始值
    if l2.val < l1.val:
        l1, l2 = l2, l1
    # 保存一个链表头用来作为返回值
    head = l1
    # 开始迭代到l1为最后一个节点
    while l1.next is not None:
        # 假如l2完结，工作完成
        if l2 is None:
            return head
        # 假如l2节点属于在l1的当前节点与下一个节点值之间
        if l1.val <= l2.val <= l1.next.val:
            # 在这一步我们通过设置l1.next\l2.next来拼接l2，并将L2 迭代
            l1.next, l2.next, l2 = l2, l1.next, l2.next
        # l1迭代向前
        l1 = l1.next
    # 以防l2较长的情况，我们在l1迭代完成后把l2加入到l1尾部
    l1.next = l2
    return head
``````

测试代码

``````python
if __name__ == '__main__':

    three = Node()
    three.val = 3

    two = Node()
    two.val = 2
    two.next = three

    one = Node()
    one.val = 1
    one.next = two

    head = Node()
    head.val = 0
    head.next = one

    three1 = Node()
    three1.val = 3

    two1 = Node()
    two1.val = 2
    two1.next = three1

    one1 = Node()
    one1.val = 1
    one1.next = two1

    head1 = Node()
    head1.val = 0
    head1.next = one1
    newhead = mergeTwoLists(head, head1)
    while newhead:
        print(newhead.val, )
        newhead = newhead.next
``````

