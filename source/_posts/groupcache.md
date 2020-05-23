---
title: Groupcache源码阅读（一）——LRU淘汰策略实现
date: 2020-05-23T14:48:26+08:00
tags:
 - goalng
 - 缓存
 - LRU

---



这里是Groupcache源码阅读计划的第一步，首先我们熟悉一下这个项目的代码结构。
<!-- more -->

## 源代码结构树

``````ini
groupcache-master
 ├── byteview.go
 ├── byteview_test.go
 ├── consistenthash
 │   ├── consistenthash.go
 │   └── consistenthash_test.go
 ├── groupcache.go
 ├── groupcachepb
 │   ├── groupcache.pb.go
 │   └── groupcache.proto
 ├── groupcache_test.go
 ├── http.go
 ├── http_test.go
 ├── LICENSE
 ├── lru
 │   ├── lru.go
 │   └── lru_test.go
 ├── peers.go
 ├── README.md
 ├── singleflight
 │   ├── singleflight.go
 │   └── singleflight_test.go
 ├── sinks.go
 └── testpb
     ├── test.pb.go
     └── test.proto
``````

## 工作流程

![YjjarR.png](https://s1.ax1x.com/2020/05/23/YjjarR.png)

## Groupcache的缓存淘汰策略

涉及到缓存，那么缓存的淘汰策略必须是要首先考虑的，下面介绍一下几种常见的缓存淘汰策略：

### FIFO(First In First Out)

先进先出，也就是淘汰缓存中最老(最早添加)的记录。FIFO 认为，最早添加的记录，其不再被使用的可能性比刚添加的可能性大。这种算法的实现也非常简单，创建一个队列，新增记录添加到队尾，每次内存不够时，淘汰队首。但是很多场景下，部分记录虽然是最早添加但也最常被访问，而不得不因为呆的时间太长而被淘汰。这类数据会被频繁地添加进缓存，又被淘汰出去，导致缓存命中率降低。

### LFU(Least Frequently Used)

最少使用，也就是淘汰缓存中访问频率最低的记录。LFU 认为，如果数据过去被访问多次，那么将来被访问的频率也更高。LFU 的实现需要维护一个按照访问次数排序的队列，每次访问，访问次数加1，队列重新排序，淘汰时选择访问次数最少的即可。LFU 算法的命中率是比较高的，但缺点也非常明显，维护每个记录的访问次数，对内存的消耗是很高的；另外，如果数据的访问模式发生变化，LFU 需要较长的时间去适应，也就是说 LFU 算法受历史数据的影响比较大。例如某个数据历史上访问次数奇高，但在某个时间点之后几乎不再被访问，但因为历史访问次数过高，而迟迟不能被淘汰。

### LRU(Least Recently Used)

最近最少使用，相对于仅考虑时间因素的 FIFO 和仅考虑访问频率的 LFU，LRU 算法可以认为是相对平衡的一种淘汰算法。LRU 认为，如果数据最近被访问过，那么将来被访问的概率也会更高。LRU 算法的实现非常简单，维护一个队列，如果某条记录被访问了，则移动到队尾，那么队首则是最近最少访问的数据，淘汰该条记录即可。

具体实现原理如下

[![Yv8RZd.png](https://s1.ax1x.com/2020/05/23/Yv8RZd.png)](https://imgchr.com/i/Yv8RZd)

### `Groupcache`中的LRU实现

![Yvp338.jpg](https://s1.ax1x.com/2020/05/23/Yvp338.jpg)

这张图很好地表示了 LRU 算法最核心的 2 个数据结构

- 蓝色的是字典(map)，存储键和值的映射关系。这样根据某个键(key)查找对应的值(value)的复杂是`O(1)`，在字典中插入一条记录的复杂度也是`O(1)`。
- 红色的是双向链表(double linked list)实现的队列。将所有的值放到双向链表中，这样，当访问到某个值时，将其移动到队尾的复杂度是`O(1)`，在队尾新增一条记录以及删除一条记录的复杂度均为`O(1)`。

### 核心数据结构：

``````go
package lru

import "container/list"

// Cache is an LRU cache. It is not safe for concurrent access.
// groupcache的核心数据结构
type Cache struct {
	// MaxEntries is the maximum number of cache entries before
	// an item is evicted. Zero means no limit.

	MaxEntries int 	// maxBytes 是允许使用的最大内存

	// OnEvicted optionally specifies a callback function to be
	// executed when an entry is purged from the cache.
	OnEvicted func(key Key, value interface{}) //提供一个淘汰值时的钩子函数

	
	ll    *list.List // 用于实现LRU的双向链表
	cache map[interface{}]*list.Element // 键是空接口，值是双向链表中对应节点的指针。
}

// A Key may be any value that is comparable. See http://golang.org/ref/spec#Comparison_operators
type Key interface{} //只要是可以用来作为比较的对象，均可以作为groupcache的key

// 键值对 entry 是双向链表节点的数据类型，在链表中仍保存每个值对应的 key 的好处在于，淘汰队首节点时，需要用 key 从字典中删除对应的映射
type entry struct {
	key   Key
	value interface{}
}

// New creates a new Cache.
// If maxEntries is zero, the cache has no limit and it's assumed
// that eviction is done by the caller.
// 方便实例化 Cache，实现 New() 函数
func New(maxEntries int) *Cache {
	return &Cache{
		MaxEntries: maxEntries,
		ll:         list.New(),
		cache:      make(map[interface{}]*list.Element),
	}
}
``````

后续就是对这个数据结构常规的增删改查：

### 查找

``````go
// Get looks up a key's value from the cache.
// 第一步是从字典中找到对应的双向链表的节点，第二步，将该节点移动到队尾
// 如果键对应的链表节点存在，则将对应节点移动到队尾，并返回查找到的值。
// c.ll.MoveToFront(ele)，即将链表中的节点 ele 移动到队尾（双向链表作为队列，队首队尾是相对的，在这里约定 front 为队尾）
func (c *Cache) Get(key Key) (value interface{}, ok bool) {
	if c.cache == nil {
		return
	}
	if ele, hit := c.cache[key]; hit {
		c.ll.MoveToFront(ele)
		return ele.Value.(*entry).value, true
	}
	return
}

``````

### 增加/修改

``````go
// Add adds a value to the cache.
//如果键存在，则更新对应节点的值，并将该节点移到队尾。
// 不存在则是新增场景，首先队尾添加新节点 &entry{key, value}, 并字典中添加 key 和节点的映射关系。
// 更新 c.nbytes，如果超过了设定的最大值 c.maxBytes，则移除最少访问的节点。
func (c *Cache) Add(key Key, value interface{}) {
	if c.cache == nil {
		c.cache = make(map[interface{}]*list.Element)
		c.ll = list.New()
	}
	if ee, ok := c.cache[key]; ok {
		c.ll.MoveToFront(ee)
		ee.Value.(*entry).value = value
		return
	}
	ele := c.ll.PushFront(&entry{key, value})
	c.cache[key] = ele
	if c.MaxEntries != 0 && c.ll.Len() > c.MaxEntries {
		c.RemoveOldest()
	}
}
``````

### 删除

- 删除指定key

``````go
// Remove removes the provided key from the cache.
func (c *Cache) Remove(key Key) {
if c.cache == nil {
	return
}
if ele, hit := c.cache[key]; hit {
	c.removeElement(ele)
}
}
``````

  

- lru删除最近使用频率最低的key

``````go
// RemoveOldest removes the oldest item from the cache.
func (c *Cache) RemoveOldest() {
if c.cache == nil {
	return
}
ele := c.ll.Back()
if ele != nil {
	c.removeElement(ele)
}
}
``````

移除元素

``````go

func (c *Cache) removeElement(e *list.Element) {
	c.ll.Remove(e)
	kv := e.Value.(*entry)
	delete(c.cache, kv.key)
	if c.OnEvicted != nil {
		c.OnEvicted(kv.key, kv.value)
	}
}
``````

### 其它

``````go
// Len returns the number of items in the cache.
func (c *Cache) Len() int {
	if c.cache == nil {
		return 0
	}
	return c.ll.Len()
}

// Clear purges all stored items from the cache.
func (c *Cache) Clear() {
	if c.OnEvicted != nil {
		for _, e := range c.cache {
			kv := e.Value.(*entry)
			c.OnEvicted(kv.key, kv.value)
		}
	}
	c.ll = nil
	c.cache = nil
}

``````

源码注释链接：

[源码注释](https://github.com/hyyc554/groupcache-coderead/blob/master/lru/lru.go)



## 参考资料

> https://geektutu.com/post/geecache-day1.html
>
> https://github.com/golang/groupcache
>
> https://blog.csdn.net/zhu592665411/article/details/80617081