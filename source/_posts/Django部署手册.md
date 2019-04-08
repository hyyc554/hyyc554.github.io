---
title: Django部署手册
date: 2018-11-25 17:25:00
tags:
  - Django
---



## 一、 安装python环境

详情参考我的这一篇博文：

<!--more-->

[Python虚拟环境配置](https://hyyc554.github.io/2018/10/26/pythonenv/)

## 二、Django的配置

```bash
pip install Django
django-admin.py startproject mysite
cd mysite
```

我将在mysite目录下完成后续相关操作

## 二、uWSGI的安装

```
pip install uwsgi
```

### 1. 基础测试：

1. 创建一个在mysite下创建一个`test.py`的测试文档

```bash
(dj11.7) [root@localhost ~]# mkdir myporject
(dj11.7) [root@localhost ~]# cd myporject/
(dj11.7) [root@localhost myporject]# django-admin startproject mysite
(dj11.7) [root@localhost myporject]# cd mysite
(dj11.7) [root@localhost mysite]# ls
manage.py  mysite
(dj11.7) [root@localhost mysite]# touch test.py
(dj11.7) [root@localhost mysite]# ls
manage.py  mysite  test.py
```

​	以上就完成了测试脚本文件的构建

1. 在`test.py`中写入以下测试内容:

```python
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2
```

1. uWSGI运行:

```
uwsgi --http :8000 --wsgi-file test.py
```

​	选项的意思是:

- `http :8000 `:使用协议http端口8000
- `wsgi-file test.py `:test.py加载指定的文件

1. 浏览器访问你的IP加端口8000。

```
http://yourIP:8000
```

​	返回结果：

```
hello world
```

如果是这样,这意味着以下工作原理:

```
the web client <-> uWSGI <-> Python
```





### 2. 测试Django项目 

现在我们希望uWSGI做同样的事情,但运行Django网站而不是 `test.py `模块。

如果您还没有这样做,确保你的 `mysite `项目实际工作原理:

```
python manage.py runserver 0.0.0.0:8000
```

如果这工作,运行它使用uWSGI:

```
uwsgi --http :8000 --module mysite.wsgi
```

- `模块 mysite.wsgi `:加载指定wsgi模块

您的浏览器指向服务器; 如果网站出现,这意味着uWSGI能够，大概的页面如下

![Fk8F61.png](https://s1.ax1x.com/2018/11/25/Fk8F61.png)

这个栈操作 正确:

```
the web client <-> uWSGI <-> Django
```

现在我们通常不会有浏览器直接向uWSGI说话。 这是一份工作 的网络服务器,它将充当中间人。





nginx。。。待续  

