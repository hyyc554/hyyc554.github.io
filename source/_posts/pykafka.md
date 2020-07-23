---
title: Python使用协程库aiokafka操作Kafka
date: 2020-06-08T20:14:03+08:00
tags:
 - python
 - kafka
---
![pyka](https://i.loli.net/2020/06/08/LeswCYIqFoD5BMh.png)
<!-- more -->
## 测试环境

由于是本地测试，我这里使用的是自己的专门用来测试的云服务器，参考我早前的博客[Docker快速搭建Kafka开发环境](https://huangyongchi.com/2019/10/31/docker-kafka/) 。

大概的情况如下：

![image-20200608195421050](https://i.loli.net/2020/06/08/mtEF6sYTD5ydH4I.png)

负载情况：

服务器上暂时没有跑其他的服务,基本跑起来一个zookeeper和两个kafka的节点，已经占去了1.5G的内存。。。所以一般也是用完就给关了

![image-20200608195319788](https://i.loli.net/2020/06/08/QtlPxbAgDn5U8XY.png)

## aiokafka

安装：

```shell
pip3 install aiokafka
```

注意：

``````shell
aiokafka 需要 kafka-python 库.
``````

## 快速开始

消费者：

``````python
from aiokafka import AIOKafkaConsumer
import asyncio

loop = asyncio.get_event_loop()

KAFKAIP = "106.53.201.23"
KAFKAPORT = 32775
async def consume():
    consumer = AIOKafkaConsumer(
        'my_topic', 'my_other_topic',
        loop=loop, bootstrap_servers=f'{KAFKAIP}:{KAFKAPORT}',
        group_id="my-group")
    # Get cluster layout and join group `my-group`
    await consumer.start()
    try:
        # Consume messages
        async for msg in consumer:
            print("consumed: ", msg.topic, msg.partition, msg.offset,
                  msg.key, msg.value, msg.timestamp)
    finally:
        # Will leave consumer group; perform autocommit if enabled.
        await consumer.stop()

loop.run_until_complete(consume())
``````

生产者：

``````python
from aiokafka import AIOKafkaProducer
import asyncio
import time

loop = asyncio.get_event_loop()

KAFKAIP = "106.53.201.23"
KAFKAPORT = 32775


async def sender(producer: AIOKafkaProducer()):
    await producer.send("my_topic", b"Super message")


async def send_one():
    producer = AIOKafkaProducer(
        loop=loop, bootstrap_servers=f'{KAFKAIP}:{KAFKAPORT}')

    # Get cluster layout and initial topic/partition leadership information
    await producer.start()
    try:
        task_list = []
        s_time = time.time()
        for i in range(1000):
            # Produce message

            task_list.append(loop.create_task(sender(producer), ))

        await asyncio.wait(task_list)
        c_time = time.time()

        print(c_time - s_time)  # 耗时0.0579991340637207
    finally:
        # Wait for all pending messages to be delivered or expire.
        await producer.stop()


loop.run_until_complete(send_one())
``````

