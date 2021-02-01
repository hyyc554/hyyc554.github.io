---
title: sentry部署
date: 2020-12-26T17:48:59+08:00
index_img: https://s3.ax1x.com/2020/12/26/rhz878.png
tags:
 - sentry
 - 日志
---


Sentry 是一个日志收集和统计平台, 由客户端和服务端组成，目前支持大部分主流的编程语言，并提供 SDK，当程序出现异常就向服务端发送消息，服务端将消息记录到数据库中并提供一个 Web 端显示。

[![rhz878.png](https://s3.ax1x.com/2020/12/26/rhz878.png)](https://imgchr.com/i/rhz878)

## 准备工作

前置条件

- Docker 19.03.6+
- Compose 1.24.1+

硬件最低要求：

- 2G+内存

## 部署流程

### 拉取源码

做完了准备工作，就可以开始搭建 sentry 了。

从 GitHub 上面获取最新的 sentry

``````bash
git clone https://github.com/getsentry/onpremise.git
``````

### 自定义配置

个性化配置：

- `config.yml`
- `sentry.conf.py`
- `.env` w/ environment variables
- `sentry.requirements.example.txt`

预先添加一个企业微信的插件，在`sentry.requirements.example.txt`写入：

``````
sentry-wxwork
``````

邮件相关配置：

``````
###############
# Mail Server #
###############

mail.backend: 'smtp'  # Use dummy if you want to disable email entirely
mail.host: 'smtp.qq.com'
mail.port: 25
mail.username: '1415940604@qq.com'
mail.password: 'XXXXXXXXXXXX'
# mail.use-tls: fals/e
# The email address to send on behalf of
mail.from: '1415940604@qq.com'
``````

### 启动

``````
./install.sh
docker-compose up -d
``````

### 访问

127.0.0.1:9000

[![rhxNsx.png](https://s3.ax1x.com/2020/12/26/rhxNsx.png)](https://imgchr.com/i/rhxNsx)