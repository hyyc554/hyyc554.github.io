---
title: Docker构建Django部署环境（三）Docker Compose
date: 2019-05-05 18:42:08

tags:
 - Docker
categories:
 - 计算机操作系统
---
*由于Django的配置文件实在有点多，这里我们先用Flask最小化演练。*

`Docker Compose` 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。
<!-- more -->

本章将介绍 `Compose` 项目情况以及安装和使用。

`Compose` 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 `OpenStack` 中的 `Heat` 十分类似。

其代码目前在 <https://github.com/docker/compose> 上开源。

`Compose` 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

通过第一部分中的介绍，我们知道使用一个 `Dockerfile` 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

`Compose` 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 `Compose` 来进行编排管理。

## 安装Docker Compose

在Linux上安装Docker Compose

```
curl -L https://github.com/docker/compose/releases/download/1.25.0-rc1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

或者通过Pip安装Docker Compose

```shell
$ sudo pip install -U docker-compose
```

测试Docker Compose是否工作

```shell
$ docker-compose --version
```

![VX4MB4.png](https://s2.ax1x.com/2019/06/19/VX4MB4.png)

## 获取示例应用

- 应用容器，运行Python示例程序
- Redis容器，运行Redis数据库

这里我们构建一个最简单的flask服务，它将对访问计数，计数器依赖于redis的一个自增键。就是这么简单明了的一个demo。我们开始吧

创建app.py文件

```python
from flask import Flask
from redis import Redis
import os

app = Flask(__name__)
redis = Redis(host="redis", port=6379)

@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Docker Book reader! I have been seen {0} times'.format(redis.get('hits'))

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

创建requirements.txt

```
flask
redis
```

创建composeapp的Dockerfile

```
$ mkdir composeapp && cd composeapp
$ touch Dockerfile
```

在Dockerfile中写入一下内容:

``````dockerfile
FROM python:3.7
MAINTAINER James Turnbull <james@example.com>
ENV REFRESHED_AT 2016-08-01

ADD . /composeapp

WORKDIR /composeapp

RUN pip install -r requirements.txt
``````

构建composeapp镜像

```
$ sudo docker build -t jamtur01/composeapp .
```

创建docker-compose.yml文件

```ini
web:
  image: jamtur01/composeapp
  command: python app.py
  ports:
   - "5000:5000"
  volumes:
   - .:/composeapp
  links:
   - redis
redis:
  image: redis
```

启动示例应用服务，必须在docker-compose.yml文件所在目录执行

```
$ sudo docker-compose up
```

![VX4Y36.png](https://s2.ax1x.com/2019/06/19/VX4Y36.png)

访问对应的IP端口：

![VXIZOU.png](https://s2.ax1x.com/2019/06/19/VXIZOU.png)

以守护进程方式运行Compose

```
$ sudo docker-compose up -d
```

![VX4auD.png](https://s2.ax1x.com/2019/06/19/VX4auD.png)

查看服务的运行状态

```
$ sudo docker-compose ps
```

![VX4bvT.png](https://s2.ax1x.com/2019/06/19/VX4bvT.png)

查看服务的日志事件

```
$ sudo docker-compose logs
```

停止正在运行的服务

```
$ sudo docker-compose stop
```

启动这些服务

```
$ sudo docker-compose start
```

删除这些服务（必须服务停止的状态下）

```
$ sudo docker-compose rm
```