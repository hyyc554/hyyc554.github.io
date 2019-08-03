---
title: Twisted与Iot
tags:
 - Twisted
 - 网络编程
categories:
 - 计算机技术
---


![ZBrvGj.png](https://s2.ax1x.com/2019/07/07/ZBrvGj.png)
<!-- more -->
## 引子

我学习Twisted也有一段时间了，在2019年为什么我还需要这样一个古老的框架呢？原因如下：

- `asyncio`为代表的新时代下的协程框架很好，但是生态过于新，导致目前还没有可以借鉴的成熟案例。反观Twisted虽然不温不火，但是github上搜索到的开源项目源码数量还是较为可观的。
- 学习Twisted也是一个成长的过程，当熟悉了Twisted的设计理念，再来看当下流行的这些协程框架时，简直如鱼得水。
- 选择最适合的东西做事情。毫无疑问支持TCP、UDP、SSL/TLS、HTTP、IMAP、SSH、IRC以及FTP以及我司未来可能用到的MQTT协议，是最大优点
- 最后，放一点我在V站上与其他人的[讨论](https://www.v2ex.com/t/566492#reply28)，综合起来社区的建议是`Twisted`or`Golang`。所以目前也在积极的学习地鼠仔语言，说不定哪天可以用go重构这个项目，不过这已经是后话了。

## 背景知识

Twisted 是一个高性能的编程框架。Twisted是用Python实现的基于事件驱动的网络引擎框架。Twisted诞生于2000年初，在当时的网络游戏开发者看来，无论他们使用哪种语言，手中都鲜有可兼顾扩展性及跨平台的网络库。Twisted的作者试图在当时现有的环境下开发游戏，这一步走的非常艰难，他们迫切地需要一个可扩展性高、基于事件驱动、跨平台的网络开发框架，为此他们决定自己实现一个，并从那些之前的游戏和网络应用程序的开发者中学习，汲取他们的经验教训。
在不同的操作系统平台上，Twisted 利用不同的底层技术实现了高效能通信。
- 在 Windows 中， Twisted 的实现基于 VO 完成端口 CIOCP, InpuνOutputCompletion p。此）技术，它保证了底层高效地将 I/O 事件通知给框架及应用程序。
- 在 Linux 中，Twisted 的 实现基于巳poll 技术， epoll 是 Linux 下 多路复用 I/O 接口 selec印oil 的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统 CPU 利用率 。
在开发方法上， Twisted 引导程序员使用异步编程模型 。 Twisted 提供了丰富 的 Defer 、Threading 等特性来支持异步编程。这也导致了很 多开发者觉得 Twisted 难以学习，本章将深入浅出地引导读者掌握 Twisted 编程的基本方法 。

Twisted支持许多常见的传输及应用层协议，包括TCP、UDP、SSL/TLS、HTTP、IMAP、SSH、IRC以及FTP。就像Python一样，Twisted也具有“内置电池”（batteries-included）的特点。Twisted对于其支持的所有协议都带有客户端和服务器实现，同时附带有基于命令行的工具，使得配置和部署产品级的Twisted应用变得非常方便。

一般来说，每个Twisted的抽象都只与一个特定的概念相关。在学习新的Twisted抽象概念时，最需要谨记的就是：

> **多数高层次抽象都是在低层次抽象的基础上建立的，很少有另立门户的。**

因此，你在学习新的Twisted抽象概念时，始终要记住它做什么和不做什么。特别是，如果一个早期的抽象A实现了F特性，那么F特性不太可能再由其它任何抽象来实现。另外，如果另外一个抽象需要F特性，那么它会使用A而不是自己再去实现F。（通常的做法，B可能会通过继承A或获得一个指向A实例的引用）

网络非常的复杂，因此Twisted包含很多抽象的概念。通过从低层的抽象讲起，我们希望能更清楚起看到在一个Twisted程序中各个部分是怎么组织起来的。

## 核心的循环体（reactor）

第一个我们要学习的抽象，也是Twisted中最重要的，就是reactor。在每个通过Twisted搭建起来的程序中心处，不管你这个程序有多少层，总会有一个reactor循环在不停止地驱动程序的运行。再也没有比reactor提供更加基础的支持了。实际上，Twisted的其它部分（即除了reactor循环体）可以这样理解：它们都是来辅助X来更好地使用reactor，这里的X可以是提供Web网页、处理一个数据库查询请求或其它更加具体的内容。尽管坚持像上一个客户端一样使用低层APIs是可能的，但如果我们执意那样做，那么我们必需自己来实现非常多的内容。而在更高的层次上，意味着我们可以少写很多代码。

但是当在外层思考与处理问题时, 很容易就忘记了reactor的存在了。在任何一个常见大小的Twisted程序中 ，确实很少会有直接与reactor的APIs交互。低层的抽象也是一样（即我们很少会直接与其交互）。我们在上一个客户端中用到的文件描述符抽象，就被更高层的抽象更好的归纳以至于我们很少会在真正的Twisted程序中遇到。（他们在内部依然在被使用，只是我们看不到而已）

至于文件描述符抽象的消息，这并不是一个问题。让Twisted掌舵异步I/O处理，这样我们就可以更加关注我们实际要解决的问题。但对于reactor不一样，它永远都不会消失。当你选择使用Twisted，也就意味着你选择使用Reactor模式，并且意味着你需要使用回调与多任务合作的"交互式"编程方式。如果你想正确地使用Twisted，你必须牢记reactor的存在。我们将在第六部分更加详细的讲解部分内容。但是现在要强调的是：

[![ZBsKL6.png](https://s2.ax1x.com/2019/07/07/ZBsKL6.png)](https://imgchr.com/i/ZBsKL6)

我们还将用图来描述新的概念，但这两个图是需要你牢记在脑海中的。可以这样说，我在写Twisted程序时一直想着这张图。

在我们付诸于代码前，有三个新的概念需要阐述清楚：Transports, Protocols, Protocol Factories

## Transports

Transports抽象是通过Twisted中interfaces模块中ITransport接口定义的。一个Twisted的Transport代表一个可以收发字节的单条连接。对于我们的诗歌下载客户端而言，就是对一条TCP连接的抽象。但是Twisted也支持诸如Unix中管道和UDP。Transport抽象可以代表任何这样的连接并为其代表的连接处理具体的异步I/O操作细节。

如果你浏览一下ITransport中的方法，可能找不到任何接收数据的方法。这是因为Transports总是在低层完成从连接中异步读取数据的许多细节工作，然后通过回调将数据发给我们。相似的原理，Transport对象的写相关的方法为避免阻塞也不会选择立即写我们要发送的数据。告诉一个Transport要发送数据，只是意味着：尽快将这些数据发送出去，别产生阻塞就行。当然，数据会按照我们提交的顺序发送。

通常我们不会自己实现一个Transport。我们会去使用Twisted提供的实现类，即在传递给reactor时会为我们创建一个对象实例。

## Protocols

Twisted的Protocols抽象由interfaces模块中的IProtocol定义。也许你已经想到，Protocol对象实现协议内容。也就是说，一个具体的Twisted的Protocol的实现应该对应一个具体网络协议的实现，像FTP、IMAP或其它我们自己制定的协议。我们的诗歌下载协议，正如它表现的那样，就是在连接建立后将所有的诗歌内容全部发送出去并且在发送完毕后关闭连接。

严格意义上讲，每一个Twisted的Protocols类实例都为一个具体的连接提供协议解析。因此我们的程序每建立一条连接（对于服务方就是每接受一条连接），都需要一个协议实例。这就意味着，Protocol实例是存储协议状态与间断性（由于我们是通过异步I/O方式以任意大小来接收数据的）接收并累积数据的地方。

因此，Protocol实例如何得知它为哪条连接服务呢？如果你阅读IProtocol定义会发现一个makeConnection函数。这是一个回调函数，Twisted会在调用它时传递给其一个也是仅有的一个参数，即Transport实例。这个Transport实例就代表Protocol将要使用的连接。

Twisted内置了很多实现了通用协议的Protocol。你可以在[twisted.protocols.basic](http://twistedmatrix.com/trac/browser/trunk/twisted/protocols/basic.py)中找到一些稍微简单点的。在你尝试写新Protocol时，最好是看看Twisted源码是不是已经有现成的存在。如果没有，那实现一个自己的协议是非常好的，正如我们为诗歌下载客户端做的那样。

## Protocol Factories

因此每个连接需要一个自己的Protocol，而且这个Protocol是我们自己定义的类的实例。由于我们会将创建连接的工作交给Twisted来完成，Twisted需要一种方式来为一个新的连接创建一个合适的协议。创建协议就是Protocol Factories的工作了。

也许你已经猜到了，Protocol Factory的API由[IProtocolFactory](http://twistedmatrix.com/trac/browser/trunk/twisted/internet/interfaces.py)来定义，同样在[interfaces](http://twistedmatrix.com/trac/browser/trunk/twisted/internet/interfaces.py)模块中。Protocol Factory就是Factory模式的一个具体实现。buildProtocol方法在每次被调用时返回一个新Protocol实例，它就是Twisted用来为新连接创建新Protocol实例的方法。