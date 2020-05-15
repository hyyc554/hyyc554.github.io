---
title: k8sbug
date: 2020-05-04T17:19:39+08:00
tags:
 - k8s
 - docker
---
## 踩坑一：Kubernetes is starting
<!-- more -->
这样就大功告成了？往往事情并不会这么顺利。由于众所周知的原因，启动 Kubernetes 所需的镜像往往会下载失败，于是点击 *Apply* 后，该配置页面的右下角始终显示 *Kubernetes is starting*，无法正常启动。

然后确认一下 Docker Desktop 自带的 Kubernetes 的版本。点击 Docker 图标，选择 *About Docker Desktop*，看到如下界面：

![YC1jVx.png](https://s1.ax1x.com/2020/05/04/YC1jVx.png)



可以看到 Kubernetes 的版本是 v1.15.5。

[k8s-for-docker-desktop](https://github.com/AliyunContainerService/k8s-for-docker-desktop)中对应的版本也应该是1.15.5.


确保文件中的 Kubernetes 版本号与 Docker Desktop 自带的 Kubernetes 版本号一致后，执行命令：

```
./load_images.sh
```

该命令会帮助我们拉取启动 Kubernetes 所需的所有镜像。命令执行完毕后，点击 Docker 图标，在 *Preferences.. > Reset* 界面中点击 *Reset Kubernetes cluster*，重启 Kubernetes。大功告成！

## 踩坑二：安装 Dashboard

[Kubernetes Dashboard](https://github.com/kubernetes/dashboard) 是 Kubernetes 集群可视化的仪表盘。

一般来说我们直接通过一行 kubectl 命令进行安装就好了：

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
复制代码
```

但由于众所周知的原因，镜像还是会下载失败，pod 始终显示 *ImagePullBackOff*。这需要我们手动拉取所需镜像。

### 下载 yaml 文件

先把 yaml 配置文件下载下来：

```
$ curl -O https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```



### 重新安装 Dashboard

如果刚才你已经执行了：

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

那么先把这个启动的 pod 删除：

```
$ kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

接着使用我们修改过的配置文件重新安装 Dashboard：

```
$ kubectl delete -f kubernetes-dashboard.yaml
```

### 启动 Dashboard 并访问

使用 kubectl 命令启动 Dashboard：

```
$ kubectl proxy
```

启动成功后，可以通过该地址进行访问 Dashboard：

> 作者：江不知
> 链接：https://juejin.im/post/5d87980f5188253f74438bb6
> 来源：掘金
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。