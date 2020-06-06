---
title: Docker构建Django部署环境（二）基本指令集
date: 2019-05-05 18:42:08

tags:
 - Docker
categories:
 - 计算机操作系统
---

## 容器相关指令
<!-- more -->
docker 命令采用了分组管理的思想，已经纳入管理的docker命令如下(版本18.09.2)： 

``````shell
Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  engine      Manage the docker engine
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes
``````

其中docker container 则是管理容器命令，老的版本中是使用docker进行容器管理，新版本兼容老版本docker命令，所以对容器管理既可用docker也可以用docker container。（一般示例代码汇总都是直接使用docker指令）

### 1.启动容器 

```shell
docker run [OPTIONS] IMAGE [COMMAND][ARG…]
```

常用OPTIONS：

- -i：		       --interactive,交互式启动
- -t：	                --tty，分配终端
- -v：                 --volume,挂在数据卷
- -d：                 --detach，后台运行
- --name：        容器名字
- --network：   指定网络
- --rm：             容器停止自动删除容器
- -P：			 自动暴露所有容器内端口，宿主随机分配端口
- -p：			 指定端口映射，将容器内服务的端口映射到宿主机的指定端口，可以使用多个-p
  - 可以使用如下三种方式：
    - `<container port>`：随机分配宿主机的一个端口作为映射端口
    - `<hostport>:<container port>`:指明主机的端口映射为容器端口
    - `<hostip>:<hostport>:<container port>`:指定主机ip和端口

示例：运行一个名字为nginx-container的容器，使用镜像nginx，并将宿主机的8080映射到容器内部80端口，然后进入交互模式。 

```
[root@app51 ~]# docker run -it --name nginx-container -p 8080:80  nginx /bin/bash
root@fd92290433da:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

### 2.查看容器

``````
docker ps [OPTIONS]
``````

常用选项：

- -a：--all ，查看所有容器，包括退出和其他状态的
- -n:：--last int，显示最后n个创建的容器
- -l, ：--latest ，显示最近的容器

示例 :

```
root@app51 ~]# docker ps -n 2
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
4d51a1cdf4b4        busybox             "/bin/sh"                11 seconds ago      Up 9 seconds                               busybox
383f31ff8f01        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        0.0.0.0:8080->80/tcp   nginx-container
[root@app51 ~]# docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4d51a1cdf4b4        busybox             "/bin/sh"           41 seconds ago      Up 39 seconds                           busybox
[root@app51 ~]#
```

### 3.查看容器具体信息

``````
docker inspect [OPTIONS] NAME|ID [NAME|ID…]
``````

示例：

```
[root@app51 ~]# docker inspect busybox
[
    {
        "Id": "4d51a1cdf4b4e06831faa6e54a32f1f8eb544e349028083b12f5b3f87af075c9",
        "Created": "2019-02-23T09:10:20.907074902Z",
        "Path": "/bin/sh",
        "Args": [],
```

### 4.停止容器

方式一：`docker stop [OPTIONS] CONTAINER [CONTAINER…]`

方式二： `docker kill [OPTIONS] CONTAINER [CONTAINER…] `

区别：docker stop 相当于发送15停止信号，而kill是强制终止对应信号9

示例：

```
[root@app51 ~]# docker stop nginx-container 
nginx-container
```

### 5.启动已停止的容器

``````
docker start [OPTIONS] CONTAINER [CONTAINER…]
``````

常用选项：

- -a：--attach 附加终端
- -I：--interactive 交互式 

```
[root@app51 ~]# docker start -ia busybox
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # ps 
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 ps
```

### 6.删除容器

``````
docker rm [OPTIONS] CONTAINER [CONTAINER…] 
或者
docker container rm
``````

如果删除的容器正在运行则需要指定-f进行强制删除

常用选项：

- -f： --force 强制删除

示例： 

```
docker rm nginx-container
```

Ps:删除所有容器

```
docker rm -f `docker ps -a -q`
docker ps -a |awk -F ' ' '{print $1}' |xargs docker rm -f
```

### 7. 暂停某个容器

``````
docker pause CONTAINER [CONTAINER…]
``````

示例：

```
[root@app51 ~]# docker pause nginx-container
nginx-container
```

### 8.恢复暂停的容器

``````
docker unpause CONTAINER [CONTAINER…]
``````

示例

```
[root@app51 ~]# docker pause nginx-container
nginx-container
```

### 9.查看容器日志

docker logs [OPTIONS] CONTAINER

常用选项：

- -t, --timestamps ：显示日志时间

```
root@app51 ~]# docker logs nginx-container 
 10.1.201.30 - - [23/Feb/2019:10:55:33 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.109 Safari/537.36" "-"
```

### 10.在已运行的容器中运行命令

``````
docker exec [OPTIONS] CONTAINER COMMAND [ARG…]
``````

常用选项：

-   -d：--detach ，后台运行命令
-   -e, --env list             设置env
-   -i, --interactive         启用交互式
-   -t, --tty                     启用终端
-   -u, --user string        指定用户 (格式: <name|uid>[:<group|gid>])
-   -w, --workdir string       指定工作目录 

示例：

```
[root@app51 ~]# docker exec -it -u nginx nginx-container /bin/sh
$ id
uid=101(nginx) gid=101(nginx) groups=101(nginx)
$
```

### 11.容器导出

``````
docker export [OPTIONS] CONTAINER
``````

容器导出类似于容器快照,导出的是容器的在宿主机上的文件系统压缩包，导出的文件系统可使用docker import进行导入，在其他机器导入时候会以镜像的方式存在。

常用参数

- -o, --output  导出的文件名称

示例 ：

```
[root@app51 ~]# docker export nginx-container -o nginx.tar
[root@app51 ~]# ls -lh ningx.tar
-rw------- 1 root root 107M 2月  23 19:18 ningx.tar
```



### 12.将导出的容器导入为镜像

``````
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
``````

容器导入可以是文件、文件url、镜像仓库

示例： 

```
[root@app51 ~]# docker import nginx.tar nginx:v154 
sha256:fd4931710d35765edb9bbd0ea84a886e0901aa7a2de03ab2eefd9aedea0e8646
[root@app51 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               v154                fd4931710d35        10 seconds ago      108MB
<none>              <none>              940cdf68f69d        7 minutes ago       108MB
busybox             latest              d8233ab899d4        8 days ago          1.2MB
nginx               latest              f09fe80eb0e7        2 weeks ago         109MB
```

其他导入示例

```
docker import http://example.com/image.tar.gz  repository:tag  
```

### 13.将容器提交为镜像

``````
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
``````

常用选项：

- -a,--author     指定作者
- -m, --message 本次提交的信息
-  -p, --pause      提交为镜像时候暂停容器
- -c, --change list 修改镜像某些属性，列如启动命令

示例： 

```
[root@app51 ~]# docker commit -p -m 'build nginx image' nginx-container nginx:test
sha256:6c68885804ca69970d747cc6cc8050ed7a1b6c24838695ec11b18348318809a6
[root@app51 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               test                6c68885804ca        6 seconds ago       109MB
nginx               v154                fd4931710d35        2 hours ago         108MB
```

## 镜像相关指令

在老版本中镜像操作也是使用的docker命令，新版本进行了分组，可使用docker image 来进行镜像操作。

### 1.搜索镜像

``````
docker search [OPTIONS] TERM
``````

常用选项：

- --limit 限制搜索的结果条目数量，默认显示25条 

```
[root@app51 ~]# docker search centos
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
centos                             The official build of CentOS.                   5179                [OK]                
ansible/centos7-ansible            Ansible on Centos7                              120                                     [OK]
jdeathe/centos-ssh                 CentOS-6 6.10 x86_64 / CentOS-7 7.5.1804 x86…   106                                     [OK]
consol/centos-xfce-vnc             Centos container with "headless" VNC session…   80                                      [OK]
```

结果字段含义：

NAME：镜像名称

DESCRIPTION :镜像描述

STARS ：获赞数量

OFFICIAL ：是否为官方镜像

AUTOMATED：是否为自动构建 

### 2.下载镜像 

``````
docker image pull  <IMAGE_NAME>:<TAG>  或者docker pull
``````

TAG不写默认为最新版本latest

```
[root@app51 ~]# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a02a4930cb5d: Pull complete
Digest: sha256:184e5f35598e333bfa7de10d8fb1cebb5ee4df5bc0f970bf2b1e7c7345136426
Status: Downloaded newer image for centos:latest
```



### 3.查看镜像

``````
docker image ls 
或者
docker images
``````

常用选项:

- -a: 查看所有已下载的镜像
- -f: --filter,过滤某些镜像 

```
[root@app51 ~]# docker image ls -a
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              1e1148e4cc2c        2 months ago        202MB
```

### 4.删除镜像

 ``````
docker image rm [OPTIONS] IMAGE [IMAGE...]  或者docker rmi IMAGE
 ``````

示例代码：

```
[root@app51 ~]# docker image rm centos
Untagged: centos:latest
Untagged: centos@sha256:184e5f35598e333bfa7de10d8fb1cebb5ee4df5bc0f970bf2b1e7c7345136426
Deleted: sha256:1e1148e4cc2c148c6890a18e3b2d2dde41a6745ceb4e5fe94a923d811bf82ddb
Deleted: sha256:071d8bd765171080d01682844524be57ac9883e53079b6ac66707e192ea25956
```

### **5. 镜像导出**

``````
docker save [OPTIONS] IMAGE [IMAGE...]
``````

将镜像打包为压缩包，可在其他docker主机进行导入,一次可打包多个

常用选项：

- -o,--output   输出到文件

示例：

```
[root@app51 ~]# docker save -o nginx-bus.tar.gz busybox:latest nginx:latest
```

### 6.镜像导入

``````
docker load [OPTIONS]
``````

将已经导出的镜像压缩文件导入为镜像

常用选项：

- -i, --input 指定文件来源 

```
[root@app51 ~]# docker load -i nginx-bus.tar.gz
Loaded image: nginx:latest
Loaded image: busybox:latest
```

### 7.查看镜像信息

``````
docker image inspect [OPTIONS] IMAGE [IMAGE...]
``````

示例代码：

```
[root@app51 ~]# docker image inspect nginx
[
    {
        "Id": "sha256:f09fe80eb0e75e97b04b9dfb065ac3fda37a8fac0161f42fca1e6fe4d0977c80",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:dd2d0ac3fff2f007d99e033b64854be0941e19a2ad51f174d9240dda20d9f534"
        ],
```

### 其他

运行信息查看docker info



```
[root@app51 ~]# docker info 
Containers: 1
 Running: 1
 Paused: 0
 Stopped: 0
Images: 4
Server Version: 18.09.2
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
```

版本信息查看 docker version

```
root@app51 ~]# docker  version
Client:
Version:           18.09.2
API version:       1.39
Go version:        go1.10.6
Git commit:        6247962
Built:             Sun Feb 10 04:13:27 2019
OS/Arch:           linux/amd64
Experimental:      false
```