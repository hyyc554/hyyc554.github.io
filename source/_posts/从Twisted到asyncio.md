---
title: 从Twisted到asyncio
date: 2019-12-26 17:37:35
tags:
 - Python
---

[Twisted](https://twistedmatrix.com/trac/)可能是Python中支持的异步编程的最古老的第三方库之一。许多开发者已使用它开发了各种应用程序。它支持许多网络协议，并可用于许多不同类型的网络编程。实际上，asyncio受Twisted的启发很大。几名专业的Twisted开发人员的也参加到了asyncio的构建工作中。不久，将会有一个基于asyncio的Twisted版本。
<!-- more -->
## 过渡

下表显示了Twisted和asyncio中一些的类似概念。

| Twisted               | asyncio                            |
| :-------------------- | :--------------------------------- |
| `Deferred`            | `asyncio.Future`                   |
| `deferToThread(func)` | `loop.run_in_executor(None, func)` |
| `@inlineCallbacks`    | `async def`                        |
| `reactor.run()`       | `loop.run_forever()`               |

## 示例

这个小例子显示了两个等效的程序，一个在Twisted中实现，一个在asyncio中实现。

- 基于 *deferred* 的 Twisted 例子：

``````python
from twisted.internet import defer
from twisted.internet import reactor


def multiply(x):
    result = x * 2
    d = defer.Deferred()
    reactor.callLater(1.0, d.callback,
                      result)
    return d


def step1(x):
    return multiply(x)


def step2(result):
    print("result: %s" % result)

    reactor.stop()


d = defer.Deferred()
d.addCallback(step1)
d.addCallback(step2)
d.callback(5)

reactor.run()
``````

- 基于asyncio的简单例子

``````python
import asyncio


async def multiply(x):
    result = x * 2
    await asyncio.sleep(1)
    return result


async def steps(x):
    result = await multiply(x)
    print("result: %s" % result)


loop = asyncio.get_event_loop()
coro = steps(5)
loop.run_until_complete(coro)
loop.close()
``````

## 参考资料

> <https://asyncio.readthedocs.io/en/latest/twisted.html>
