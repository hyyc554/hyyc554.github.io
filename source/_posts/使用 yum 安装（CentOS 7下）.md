---
title: CentOS下安装Docker
tags:
 - Linux
 - Docker
---





## 使用 yum 安装（CentOS 7下）

Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

<!--more-->

通过 `uname -r `命令查看你当前的内核版本

```
uname -r 3.10.0-327.el7.x86_64
```

![img](http://www.runoob.com/wp-content/uploads/2016/05/docker08.png)

### 安装 Docker

从 2017 年 3 月开始 docker 在原来的基础上分为两个分支版本: Docker CE 和 Docker EE。

Docker CE 即社区免费版，Docker EE 即企业版，强调安全，但需付费使用。

本文介绍 Docker CE 的安装使用。

移除旧的版本：

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

安装一些必要的系统工具：

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

添加软件源信息：

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新 yum 缓存：

```
sudo yum makecache fast
```

安装 Docker-ce：

```
sudo yum -y install docker-ce
```

启动 Docker 后台服务

```
sudo systemctl start docker
```

测试运行 hello-world

```
docker run hello-world
```

![img](http://www.runoob.com/wp-content/uploads/2016/05/docker12.png)

由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行。



## 参考资料：

> http://www.runoob.com/docker/centos-docker-install.html