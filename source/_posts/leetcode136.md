---
title: leetcode136-Single Number
date: 2020-06-06T11:39:15+08:00
tags:
 - leetcode
---


## Description

Given a **non-empty** array of integers, every element appears *twice* except for one. Find that single one.

**Note:**

Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

**Example 1:**

```
Input: [2,2,1]
Output: 1
```

**Example 2:**

```
Input: [4,1,2,1,2]
Output: 4
```
<!-- more -->
## Solution

First of all, need to recognize follow things:

- XOR

  XOR is a binary operation, it stands for "exclusive or", that is to say the resulting bit evaluates to one if only exactly *one* of the bits is set.

  This is its function table:

  ```py
  a | b | a ^ b
  --|---|------
  0 | 0 | 0
  0 | 1 | 1
  1 | 0 | 1
  1 | 1 | 0
  ```

  This operation is performed between every two corresponding bits of a number.

  Example: `7 ^ 10`
  In binary: `0111 ^ 1010`

  ```py
    0111
  ^ 1010
  ======
    1101 = 13
  ```

  **Properties:** The operation is commutative, associative and self-inverse.

- XOR rules

  - any number XOR to itself return 0
  - any number XOR to 0 return itself 

python code:

``````python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        single_number = 0
        for num in nums:
            single_number ^= num
        return single_number
``````

Golang code:

``````go
func singleNumber(nums []int) int {
    s_num := 0
    for _,num := range nums{
        s_num ^= num
    }
    return s_num
}
``````
