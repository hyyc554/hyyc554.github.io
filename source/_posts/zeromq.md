---
title: ZeroMQ自查手册
tags:
 - 消息队列
 - 网络编程
categories:
 - 计算机技术
---

## 简介

> ZMQ (以下 ZeroMQ 简称 ZMQ)是一个简单好用的传输层，像框架一样的一个 socket library，他使得 Socket 编程更加简单、简洁和性能更高。是一个消息处理队列库，可在多个线程、内核和主机盒之间弹性伸缩。ZMQ 的明确目标是“成为标准网络协议栈的一部分，之后进入 Linux 内核”。现在还未看到它们的成功。但是，它无疑是极具前景的、并且是人们更加需要的“传统”BSD 套接字之上的一层封装。ZMQ 让编写高性能网络应用程序极为简单和有趣。

![ZgrBBd.jpg](https://s2.ax1x.com/2019/07/10/ZgrBBd.jpg)
<!-- more -->
`RabbitMQ`是一个AMQP实现，传统的messaging queue系统实现，基于Erlang。老牌MQ产品了。AMQP协议更多用在企业系统内，对数据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量还在其次。

`Kafka`是linkedin开源的MQ系统，主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志收集和传输，0.8开始支持复制，不支持事务，适合产生大量数据的互联网服务的数据收集业务。

`ZeroMQ`只是一个网络编程的Pattern库，将常见的网络请求形式（分组管理，链接管理，发布订阅等）模式化、组件化，简而言之socket之上、MQ之下。对于MQ来说，网络传输只是它的一部分，更多需要处理的是消息存储、路由、Broker服务发现和查找、事务、消费模式（ack、重投等）、集群服务等。

综上所述，`Zeromq` 并不是类似`Rabbitmq`消息列队，它实际上只一个消息列队组件，一个库。



## Zeromq的几种模式

### Request-Reply模式：

客户端在请求后，服务端必须回响应
![ZgU6Qs.png](https://s2.ax1x.com/2019/07/10/ZgU6Qs.png)


Python实现：
server端：

```python
# -*- coding=utf-8 -*-
import zmq

context = zmq.Context()
socket = context.socket(zmq.REP)
socket.bind("tcp://*:5555")

while True:
    message = socket.recv()
    print("Received: %s" % message)
    socket.send("I am OK!")
```

client端：

```python
# -*- coding=utf-8 -*-

import zmq
import sys

context = zmq.Context()
socket = context.socket(zmq.REQ)
socket.connect("tcp://localhost:5555")

socket.send('Are you OK?')
response = socket.recv();
print("response: %s" % response)
```

输出：

```python
$ python app/server.py 
Received: Are you OK?

$ python app/client1.py 
response: I am OK!
```

### Publish-Subscribe模式:

广播所有client，没有队列缓存，断开连接数据将永远丢失。client可以进行数据过滤。

![ZgUrWQ.png](https://s2.ax1x.com/2019/07/10/ZgUrWQ.png)



Python实现
server端：

```python
# -*- coding=utf-8 -*-
import zmq
import time

context = zmq.Context()
socket = context.socket(zmq.PUB)
socket.bind("tcp://*:5555")

while True:
    print('发送消息')
    socket.send("消息群发")
    time.sleep(1)    
```

client端1：

```python
# -*- coding=utf-8 -*-

import zmq
import sys

context = zmq.Context()
socket = context.socket(zmq.SUB)
socket.connect("tcp://localhost:5555")
socket.setsockopt(zmq.SUBSCRIBE,'')  # 消息过滤
while True:
    response = socket.recv();
    print("response: %s" % response)
```

client端2：

```python
# -*- coding=utf-8 -*-

import zmq
import sys

context = zmq.Context()
socket = context.socket(zmq.SUB)
socket.connect("tcp://localhost:5555")
socket.setsockopt(zmq.SUBSCRIBE,'') 
while True:
    response = socket.recv();
    print("response: %s" % response)
```

输出：

```python
$ python app/server.py 
发送消息
发送消息
发送消息

$ python app/client2.py 
response: 消息群发
response: 消息群发
response: 消息群发

$ python app/client1.py 
response: 消息群发
response: 消息群发
response: 消息群发
```

### Parallel Pipeline模式：

由三部分组成，push进行数据推送，work进行数据缓存，pull进行数据竞争获取处理。区别于Publish-Subscribe存在一个数据缓存和处理负载。

当连接被断开，数据不会丢失，重连后数据继续发送到对端。
![ZgUszj.png](https://s2.ax1x.com/2019/07/10/ZgUszj.png)

Python实现

server端：

```python
# -*- coding=utf-8 -*-
import zmq
import time

context = zmq.Context()
socket = context.socket(zmq.PUSH)
socket.bind("tcp://*:5557")

while True:
    socket.send("测试消息")
    print "已发送"    
    time.sleep(1)    
```

work端：

```python
# -*- coding=utf-8 -*-

import zmq

context = zmq.Context()

recive = context.socket(zmq.PULL)
recive.connect('tcp://127.0.0.1:5557')

sender = context.socket(zmq.PUSH)
sender.connect('tcp://127.0.0.1:5558')

while True:
    data = recive.recv()
    print "正在转发..."
    sender.send(data)
```

client端：

```python
# -*- coding=utf-8 -*-

import zmq
import sys

context = zmq.Context()
socket = context.socket(zmq.PULL)
socket.bind("tcp://*:5558")

while True:
    response = socket.recv();
    print("response: %s" % response)
```

输出结果：



```python
$ python app/server.py 
已发送
已发送
已发送

$ python app/work.py 
正在转发...
正在转发...
正在转发...

$ python app/client1.py
response: 测试消息
response: 测试消息
response: 测试消息
```