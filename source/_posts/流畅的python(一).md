---
title: 流畅的Python读书笔记(1)
tags:
  - 读书笔记
  - Python
---
流畅的Python读书笔记

## 关于

> 前言
>
> Python 官方教程（https://docs.python.org/3/tutorial/）的开头是这样写的： “Python 是一门既
> 容易上手又强大的编程语言。 ”这句话本身并无大碍，但需要注意的是，正因为它既好学
> 又好用，所以很多 Python 程序员只用到了其强大功能的一小部分。
<!-- more -->
工作中人们往往带着一种这很简单的错觉，希望通过这本书我可以学习到进阶的知识，摆脱这样的印象

## 第一章 Python数据模型



1.1 python风格的纸牌

``````python
import collections
Card = collections.namedtuple('Card', ['rank', 'suit'])
class FrenchDeck:
	ranks = [str(n) for n in range(2, 11)] + list('JQKA')
	suits = 'spades diamonds clubs hearts'.split()
	def __init__(self):
		self._cards = [Card(rank, suit) for suit in self.suits 
                   					for rank in self.ranks]
	def __len__(self):
        return len(self._cards)
	def __getitem__(self, position):
		return self._cards[position]
``````



关于`collections`的用法

> https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431953239820157155d21c494e5786fce303f3018c86000



 