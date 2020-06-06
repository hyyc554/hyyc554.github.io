---
title: groupcache源码阅读(二）——缓存并发控制
date: 2020-05-24T17:44:23+08:00
tags:
    - Golang
---

![t9CtxI.jpg](https://s1.ax1x.com/2020/05/25/t9CtxI.jpg)

## 互斥锁——sync.Mutex

多个协程(goroutine)同时读写同一个变量，在并发度较高的情况下，会发生冲突。确保一次只有一个协程(goroutine)可以访问该变量以避免冲突，这称之为`互斥`，互斥锁可以解决这个问题。

> sync.Mutex 是一个互斥锁，可以由不同的协程加锁和解锁。

`sync.Mutex` 是 Go 语言标准库提供的一个互斥锁，当一个协程(goroutine)获得了这个锁的拥有权后，其它请求锁的协程(goroutine) 就会阻塞在 `Lock()` 方法的调用上，直到调用 `Unlock()` 锁被释放。

## Groupcache的并发数据对象

### `ByteView`

对于Groupcache这种key、value的缓存，value再上一篇文章中显示了是一个空接口。而这里用`ByteView` 明确了，value的具体含义。

只读数据结构 `ByteView` 用来表示缓存值，是 GeeCache 主要的数据结构之一

``````go
// A ByteView holds an immutable view of bytes.
// Internally it wraps either a []byte or a string,
// but that detail is invisible to callers.
//
// A ByteView is meant to be used as a value type, not
// a pointer (like a time.Time).
// 一个抽象的制度数据结构，用来保存缓存值
// ByteView 有两个数据成员，
// 			- b []byte，b 将会存储真实的缓存值。选择 byte 类型是为了能够支持任意的数据类型的存储，例如图片、视频等。
// 			- s string .s将会用来直接保存字符串
type ByteView struct {
	// If b is non-nil, b is used, else s is used.
	b []byte
	s string
}
``````

ByteView对象配套的增删改查

``````go
// ByteSlice returns a copy of the data as a byte slice.
// b 是只读的，使用 ByteSlice() 方法返回一个拷贝，防止缓存值被外部程序修改。
func (v ByteView) ByteSlice() []byte {
	if v.b != nil {
		return cloneBytes(v.b)
	}
	return []byte(v.s)
}
``````

### cache再度封装lru.Cache 

在[groupcache.go](https://github.com/hyyc554/groupcache-coderead/blob/master/groupcache.go)中定义的cache是对底层lur的并发封装

``````go
// cache is a wrapper around an *lru.Cache that adds synchronization,
// makes values always be ByteView, and counts the size of all keys and
// values.
// 对lru.Cache进行并发控制
type cache struct {
	mu         sync.RWMutex
	nbytes     int64 // of all keys and values
	lru        *lru.Cache
	nhit, nget int64
	nevict     int64 // number of evictions
}
``````

对应的增删改查：

``````go
/*
在 add 方法中，判断了 c.lru 是否为 nil，如果不等于 nil 再创建实例。
这种方法称之为延迟初始化(Lazy Initialization)，
一个对象的延迟初始化意味着该对象的创建将会延迟至第一次使用该对象时。
主要用于提高性能，并减少程序内存要求
*/
func (c *cache) add(key string, value ByteView) {
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.lru == nil {
		c.lru = &lru.Cache{
			OnEvicted: func(key lru.Key, value interface{}) {
				val := value.(ByteView)
				c.nbytes -= int64(len(key.(string))) + int64(val.Len())
				c.nevict++
			},
		}
	}
	c.lru.Add(key, value)
	c.nbytes += int64(len(key)) + int64(value.Len())
}

func (c *cache) get(key string) (value ByteView, ok bool) {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.nget++
	if c.lru == nil {
		return
	}
	vi, ok := c.lru.Get(key)
	if !ok {
		return
	}
	c.nhit++
	return vi.(ByteView), true
}
``````

### 主体结构 Group

Group 是 GroupCache 最核心的数据结构，负责与用户的交互，并且控制缓存值存储和获取的流程。

具体流程如下：

![](https://s1.ax1x.com/2020/05/23/YjjarR.png)

上述的过程均在group对象中完成，以下是group的结构体:

``````go
// A Group is a cache namespace and associated data loaded spread over
// a group of 1 or more machines.
type Group struct {
	name       string  // Group的名字
	// 定义接口 Getter 和 回调函数 Get(key string)([]byte, error)，参数是 key，返回值是 []byte。
	// getter相当于是group获取本地、远端数据的接口方法
	getter     Getter // 即缓存未命中时获取源数据的回调(callback)
	peersOnce  sync.Once	// 单例化
	peers      PeerPicker // 节点选择方法
	cacheBytes int64 // limit for sum of mainCache and hotCache size

	// mainCache is a cache of the keys for which this process
	// (amongst its peers) is authoritative. That is, this cache
	// contains keys which consistent hash on to this process's
	// peer number.
	mainCache cache // 当前group维持的内存数据对象，并发缓存

	// hotCache contains keys/values for which this peer is not
	// authoritative (otherwise they would be in mainCache), but
	// are popular enough to warrant mirroring in this process to
	// avoid going over the network to fetch from a peer.  Having
	// a hotCache avoids network hotspotting, where a peer's
	// network card could become the bottleneck on a popular key.
	// This cache is used sparingly to maximize the total number
	// of key/value pairs that can be stored globally.
	hotCache cache // 简而言之就是，对不在本节点的热点数据，进行本地缓存，以免大量的网络io

	// loadGroup ensures that each key is only fetched once
	// (either locally or remotely), regardless of the number of
	// concurrent callers.
	loadGroup flightGroup //处理惊群效应

	_ int32 // force Stats to be 8-byte aligned on 32-bit platforms

	// Stats are statistics on the group.
	Stats Stats // group状态维持情况
}
``````

对Group的实例化：

``````go
// NewGroup creates a coordinated group-aware Getter from a Getter.
//
// The returned Getter tries (but does not guarantee) to run only one
// Get call at once for a given key across an entire set of peer
// processes. Concurrent callers both in the local process and in
// other processes receive copies of the answer once the original Get
// completes.
//
// The group name must be unique for each getter.
// 构建函数 NewGroup 用来实例化 Group，并且将 group 存储在全局变量 groups 中
func NewGroup(name string, cacheBytes int64, getter Getter) *Group {
	return newGroup(name, cacheBytes, getter, nil)
}

// If peers is nil, the peerPicker is called via a sync.Once to initialize it.
func newGroup(name string, cacheBytes int64, getter Getter, peers PeerPicker) *Group {
	if getter == nil {
		panic("nil Getter")
	}
	mu.Lock()
	defer mu.Unlock()
	initPeerServerOnce.Do(callInitPeerServer)
	if _, dup := groups[name]; dup {
		panic("duplicate registration of group " + name)
	}
	g := &Group{
		name:       name,
		getter:     getter,
		peers:      peers,
		cacheBytes: cacheBytes,
		loadGroup:  &singleflight.Group{},
	}
	if fn := newGroupHook; fn != nil {
		fn(g)
	}
	groups[name] = g
	return g
}
``````

Group 的 Get 方法

``````go
// load loads key either by invoking the getter locally or by sending it to another machine.
func (g *Group) load(ctx context.Context, key string, dest Sink) (value ByteView, destPopulated bool, err error) {
	g.Stats.Loads.Add(1)
	viewi, err := g.loadGroup.Do(key, func() (interface{}, error) {
		// Check the cache again because singleflight can only dedup calls
		// that overlap concurrently.  It's possible for 2 concurrent
		// requests to miss the cache, resulting in 2 load() calls.  An
		// unfortunate goroutine scheduling would result in this callback
		// being run twice, serially.  If we don't check the cache again,
		// cache.nbytes would be incremented below even though there will
		// be only one entry for this key.
		//
		// Consider the following serialized event ordering for two
		// goroutines in which this callback gets called twice for the
		// same key:
		// 1: Get("key")
		// 2: Get("key")
		// 1: lookupCache("key")
		// 2: lookupCache("key")
		// 1: load("key")
		// 2: load("key")
		// 1: loadGroup.Do("key", fn)
		// 1: fn()
		// 2: loadGroup.Do("key", fn)
		// 2: fn()
		if value, cacheHit := g.lookupCache(key); cacheHit {
			g.Stats.CacheHits.Add(1)
			return value, nil
		}
		g.Stats.LoadsDeduped.Add(1)
		var value ByteView
		var err error
		// 进行节点选择，是读取本地数据，还是从远端节点获取数据
		if peer, ok := g.peers.PickPeer(key); ok {
			value, err = g.getFromPeer(ctx, peer, key)
			if err == nil {
				g.Stats.PeerLoads.Add(1)
				return value, nil
			}
			g.Stats.PeerErrors.Add(1)
			// TODO(bradfitz): log the peer's error? keep
			// log of the past few for /groupcachez?  It's
			// probably boring (normal task movement), so not
			// worth logging I imagine.
		}
		value, err = g.getLocally(ctx, key, dest)
		if err != nil {
			g.Stats.LocalLoadErrs.Add(1)
			return nil, err
		}
		g.Stats.LocalLoads.Add(1)
		destPopulated = true // only one caller of load gets this return value
		g.populateCache(key, value, &g.mainCache)
		return value, nil
	})
	if err == nil {
		value = viewi.(ByteView)
	}
	return
}

func (g *Group) getLocally(ctx context.Context, key string, dest Sink) (ByteView, error) {
	err := g.getter.Get(ctx, key, dest)
	if err != nil {
		return ByteView{}, err
	}
	return dest.view()
}

``````

- Get 方法实现了上述所说的流程 。
- 流程 1：从 mainCache 中查找缓存，如果存在则返回缓存值。
- 流程 2：节点选择，判断是本地还是远端节点
- 流程 3：缓存不存在，则调用 load 方法，load 调用 getLocally（分布式场景下会调用 getFromPeer 从其他节点获取），getLocally 调用用户回调函数 `g.getter.Get()` 获取源数据，并且将源数据添加到缓存 mainCache 中（通过 populateCache 方法）

## 总结

通过上面的代码已经可以看出Group单节点内，对于并发请求缓存数据，实际上就是加入了一个互斥锁，已经对缓存数据进行只读拷贝，确保读取的安全。

