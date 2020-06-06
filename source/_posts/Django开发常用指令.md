---
title: Django开发常用指令
date: 2019-05-05 18:42:08

tags:
 - Python
 - django

---

## Django相关指令
<!-- more -->
django-admin.py和manage.py这两个文件代码和包含命令基本是一样的，只不过django-admin.py一般只用来创建项目，而manage.py用来管理创建好了的项目。

**创建新项目**

```shell
django-admin.py startproject [project_name]
```

*注意: windows系统下请用django-admin startproject [xxx]*

**创建新应用**

```shell
python manage.py startapp [app_name]
```

*注意: 你需要先cd进入创建的项目文件夹*

**检测模型变化，生成新的数据库迁移文件**

```shell
python manage.py makemigrations [app_label]
```

*注意: app名字可选。如果一个项目包含多个app，而你只更改了其中一个app的模型，建议后面加入具体的app名*

**同步数据库与模型**

```shell
python manage.py migrate
```

**启动服务器**

```shell
python manage.py runserver 0.0.0.0:8000
```

**创建超级用户**

```shell
python manage.py createsuperuser
```

**修改用户密码**

```shell
python manage.py changepassword [username]
```

**打开交互终端**

```shell
python manage.py shell
python manage.py dbshell(数据库交互)
```

**查看当前版本**

```python
python manage.py version
```

**搜集静态文件**

```shell
python manage.py collectstatic
```

**数据库备份与恢复**

1. 备份

```shell
# 备份某一个APP
python manage.py dumpdata app_name --format=json > app.json
# 备份整个db
python manage.py dumpdata --format=json > bak.json
```

1. 恢复

```shell
python manage.py loaddata app.json
```

**一些不常用的指令** 相对意义上的不常用，也可能由于笔者水平所限，暂时尚未使用过以下指令

```shell
python manage.py flush	# 清空数据库内容，只留下空表
python manage.py test	# 开始测试
python manage.py createcachetable	# 创建缓存表
python manage.py check # 检测项目有没有问题
python manage.py inspectdb [table] # 根据已有数据库反向生成django模型。你可以选择数据表名字
python manage.py makemessages # 搜集所有的messages，可以生成指定文件格式如xml文件，供后期翻译
python manage.py sendemail [email]	# 发送测试邮件
python manage.py showmigrations	# 显示所有数据库迁移文件
```



## Python相关指令

**生成requirements.txt文件**

```shell
pip freeze > requirements.txt
```

**安装requirements.txt依赖**

```shell
pip install -r requirements.txt
```

**关闭全部 Python 进程**

```shell
taskkill -f -im python
taskkill -f -im python.exe
```



## celery相关指令

**启动 celery 的后台任务**

```shell
celery -A [project_name] beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler 
```

或

```shell
python manage.py celery worker --settings=settings -l info -c 4 --autoreload
```

**启动 celery 的周期任务**

```shell
celery worker -A [project_name] -l info 
```

或者

```shell
python manage.py celery beat
```



## uwsgi相关指令

**启动**

```shell
uwsgi --ini uwsgi.ini
```

**重启**

```shell
uwsgi --reload uwsgi.pid  
```

**关闭**

```shell
uwsgi --stop uwsgi.pid
```

**强制关闭**

```shell
ps aux|grep uwsgi|awk '{print $2}'|xargs kill -9
```

**读取uwsgi实时状态**

```shell
uwsgi --connect-and-read uwsgi/uwsgi.status
```


  