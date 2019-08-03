---
title: GOPROXY 环境变量
tags:
 - Golang
categories:
 - 计算机技术
---


我们知道从 `Go 1.11` 版本开始，官方支持了 `go module` 包依赖管理工具。
<!-- more -->

其实还新增了 `GOPROXY` 环境变量。如果设置了该变量，下载源代码时将会通过这个环境变量设置的代理地址，而不再是以前的直接从代码库下载。这无疑对我等无法科学上网的开发良民来说是最大的福音。

更可喜的是，[goproxy.io](https://github.com/goproxyio/goproxy) 这个开源项目帮我们实现好了我们想要的。该项目允许开发者一键构建自己的 `GOPROXY` 代理服务。同时，也提供了公用的代理服务 `https://goproxy.io`，我们只需设置该环境变量即可正常下载被墙的源码包了：

```sh
export GOPROXY=https://goproxy.io
```

不过，**需要依赖于 go module 功能**。可通过 `export GO111MODULE=on` 开启 MODULE。

如果项目不在 `GOPATH` 中，则无法使用 `go get ...`，但可以使用 `go mod ...` 相关命令。

也可以通过置空这个环境变量来关闭，`export GOPROXY=`。

对于 Windows 用户，可以在 `PowerShell` 中设置：

```sh
$env:GOPROXY = "https://goproxy.io"
```

最后，我们当然推荐使用 `GOPROXY` 这个环境变量的解决方式，前提是 **Go version >= 1.11**。

在goland中应用：

在新建项目时：

![ZpIA00.png](https://s2.ax1x.com/2019/06/22/ZpIA00.png)