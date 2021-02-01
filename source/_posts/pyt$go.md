---
title: Python与Golang协程异同
date: 2019-05-05 18:42:08

tags:
 - Python
 - golang
categories:
 - 计算机技术
---


## 背景知识
<!-- more -->

这里先给出一些常用的知识点简要说明，以便理解后面的文章内容。

**进程的定义：**


进程，是计算机中已运行程序的实体。程序本身只是指令、数据及其组织形式的描述，进程才是程序的真正运行实例。

**线程的定义：**

操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。

**进程和线程的关系：**

一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。
CPU的最小调度单元是线程不是进程，所以单进程多线程也可以利用多核CPU.

**协程的定义：**

协程通过在线程中实现调度，避免了陷入内核级别的上下文切换造成的性能损失，进而突破了线程在IO上的性能瓶颈。

**协程和线程的关系**

协程是在语言层面实现对线程的调度，避免了内核级别的上下文消耗。

![01gjYD.jpg](https://s1.ax1x.com/2020/10/03/01gjYD.jpg)



## python协程

Python的协程源于yield指令。yield有两个功能:

- yield item用于产出一个值，反馈给next()的调用方。
- 作出让步，暂停执行生成器，让调用方继续工作，直到需要使用另一个值时再调用next()。

```python
import asyncio


async def compute(x, y):
    print("Compute %s + %s ..." % (x, y))
    await asyncio.sleep(x + y)
    return x + y


async def print_sum(x, y):
    result = await compute(x, y)
    print("%s + %s = %s" % (x, y, result))


loop = asyncio.get_event_loop()
tasks = [print_sum(1, 2), print_sum(3, 4)]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

协程是对线程的调度，yield类似惰性求值方式可以视为一种流程控制工具，实现协作式多任务,在Python3.5正式引入了` async/await`表达式，使得协程正式在语言层面得到支持和优化，大大简化之前的yield写法。
线程是内核进行抢占式的调度的，这样就确保了每个线程都有执行的机会。
而 `coroutine `运行在同一个线程中，由语言的运行时中的 `EventLoop`（事件循环）来进行调度。
和大多数语言一样，在 Python 中，协程的调度是非抢占式的，也就是说一个协程必须主动让出执行机会，其他协程才有机会运行。
让出执行的关键字就是 await。也就是说一个协程如果阻塞了，持续不让出 CPU，那么整个线程就卡住了，没有任何并发。

简而言之，任何时候只有一个协程正在运行。

*PS: 作为服务端，event loop最核心的就是IO多路复用技术，所有来自客户端的请求都由IO多路复用函数来处理;作为客户端，event loop的核心在于利用Future对象延迟执行，并使用send函数激发协程,挂起,等待服务端处理完成返回后再调用CallBack函数继续下面的流程*

## Go的协程

Go天生在语言层面支持，和Python类似都是采用了关键字，而Go语言使用了go这个关键字，可能是想表明协程是Go语言中最重要的特性。
go协程之间的通信，Go采用了channel关键字。

Go实现了两种并发形式:

- 多线程共享内存。如Java或者C++等在多线程中共享数据（例如数组、Map、或者某个结构体或对象）的时候，通过锁来访问.
- Go语言特有的，也是Go语言推荐的：CSP（communicating sequential processes）并发模型。

Go的CSP并发模型实现：M, P, G : [[https://www.cnblogs.com/sunsk...](https://www.cnblogs.com/sunsky303/p/9115530.html)]

```go
package main

import (
    "fmt"
)

//Go 协程（goroutines）和协程（coroutines）
//Go 协程意味着并行（或者可以以并行的方式部署），协程一般来说不是这样的
//Go 协程通过通道来通信；协程通过让出和恢复操作来通信

// 进程退出时不会等待并发任务结束，可用通道（channel）阻塞，然后发出退出信号
func main() {
    jobs := make(chan int)
    done := make(chan bool) // 结束标志

    go func() {
        for {
            j, more := <-jobs //  利用more这个值来判断通道是否关闭，如果关闭了，那么more的值为false，并且通知给通道done
            fmt.Println("----->:", j, more)
            if more {
                fmt.Println("received job", j)
            } else {
                fmt.Println("end received jobs")
                done <- true
                return
            }
        }
    }()

    go func() {
        for j := 1; j <= 3; j++ {
            jobs <- j
            fmt.Println("sent job", j)
        }
        close(jobs) // 写完最后的数据,紧接着就close掉
        fmt.Println("close(jobs)")
    }()

    fmt.Println("sent all jobs")
    <-done // 让main等待全部协程完成工作
}
```

通过在函数调用前使用关键字` go`，我们即可让该函数以 `goroutine` 方式执行。`goroutine`是一种 比线程更加轻盈、更省资源的协程。
Go 语言通过系统的线程来多路派遣这些函数的执行，使得 每个用 go 关键字执行的函数可以运行成为一个单位协程。
当一个协程阻塞的时候，调度器就会自 动把其他协程安排到另外的线程中去执行，从而实现了程序无等待并行化运行。
而且调度的开销非常小，一颗 CPU 调度的规模不下于每秒百万次，这使得我们能够创建大量的 `goroutine`，从而可以很轻松地编写高并发程序，达到我们想要的目的。 ---- 某书

**协程的4种状态**

- Pending
- Running
- Done
- Cacelled

**和系统线程之间的映射关系**

go的协程本质上还是系统的线程调用，而Python中的协程是eventloop模型实现，所以虽然都叫协程，但并不是一个东西.
Python 中的协程是严格的 1:N 关系，也就是一个线程对应了多个协程。虽然可以实现异步I/O，但是不能有效利用多核(GIL)。
而 Go 中是 M:N 的关系，也就是 N 个协程会映射分配到 M 个线程上，这样带来了两点好处：

- 多个线程能分配到不同核心上,CPU 密集的应用使用 goroutine 也会获得加速.
- 即使有少量阻塞的操作，也只会阻塞某个 worker 线程，而不会把整个程序阻塞。

*PS: Go中很少提及线程或进程,也就是因为上面的原因.*

## Python与Golang协程对比:

- Python中的`async`是非抢占式的,一旦开始采用 `async `函数，那么你整个程序都必须是 `async` 的，不然总会有阻塞的地方(一遇阻塞对于没有实现异步特性的库就无法主动让调度器调度其他协程了)，也就是说 `async` 具有传染性。
- Python 整个异步编程生态的问题，之前标准库和各种第三方库的阻塞性函数都不能用了，如:requests,redis.py,open 函数等。所以 Python3.5后加入协程的最大问题不是不好用，而是生态环境不好,历史包袱再次上演,动态语言基础上再加上多核之间的任务调度,应该是很难的技术吧,真心希望python4.0能优化或者放弃GIL锁,使用多核提升性能。
- `goroutine `是 Go 与生俱来的特性，所以几乎所有库都是可以直接用的，避免了 Python 中需要把所有库重写一遍的问题。
- `goroutine` 中不需要显式使用` await `交出控制权，但是 Go 也不会严格按照时间片去调度 `goroutine`，而是会在可能阻塞的地方插入调度。`goroutine` 的调度可以看做是半抢占式的。

#### PS: python异步库列表 [[https://github.com/timofurrer...](https://github.com/timofurrer/awesome-asyncio)]

------

Do not communicate by sharing memory; instead, share memory by communicating.(不要以共享内存的方式来通信，相反，要通过通信来共享内存) -- CSP并发模型

------

## 扩展与总结

erlang和golang都是采用了CSP(Communicating Sequential Processes)模式(Python中的协程是eventloop模型)
但是erlang是基于进程的消息通信，go是基于goroutine和channel的通信。
Python和Go都引入了消息调度系统模型，来避免锁的影响和进程/线程开销大的问题。
协程从本质上来说是一种用户态的线程，不需要系统来执行抢占式调度，而是在语言层面实现线程的调度。
因为协程不再使用共享内存/数据，而是使用通信来共享内存/锁，因为在一个超级大系统里具有无数的锁，
共享变量等等会使得整个系统变得无比的臃肿，而通过消息机制来交流，可以使得每个并发的单元都成为一个独立的个体，
拥有自己的变量，单元之间变量并不共享，对于单元的输入输出只有消息。
开发者只需要关心在一个并发单元的输入与输出的影响，而不需要再考虑类似于修改共享内存/数据对其它程序的影响。