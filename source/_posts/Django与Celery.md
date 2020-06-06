---
title: Django与Celery
date: 2019-05-05 18:42:08

Tags:
  - Django
  - Celery
---





先祭上 https://github.com/celery/django-celery-beat
<!--more-->

> 注意：
>
> celery4.0支持django1.8以上版本，如果1.8以下版本请使用celry3.1
>
> 另外celery4.0版本不支持windows，需要启动的时候指定下--pool=solo，下面会说明

本文从零开始，如果已有django环境和celery环境，请忽略以下步骤



# VirtualEnv环境

请参照我之前的文章-[virtualenv环境](https://my.oschina.net/u/914655/blog/1192590)

这里简单列下

```
E:\virtualenv\nowamagic_venv>virtualenv proj_env1 --python=C:\Python36\python.exe
Running virtualenv with interpreter C:\Python36\python.exe
Using base prefix 'C:\\Python36'
New python executable in E:\virtualenv\nowamagic_venv\proj_env1\Scripts\python.exe
Installing setuptools, pip, wheel...done.

E:\virtualenv\nowamagic_venv>cd proj_env1

E:\virtualenv\nowamagic_venv\proj_env1>cd Scripts

E:\virtualenv\nowamagic_venv\proj_env1\Scripts>activate

(proj_env1) E:\virtualenv\nowamagic_venv\proj_env1\Scripts>pip list
DEPRECATION: The default format will switch to columns in the future. You can use --format=(legacy|columns) (or define a
ection) to disable this warning.
pip (9.0.1)
setuptools (38.5.1)
wheel (0.30.0)
```

有时安装的pip版本太低，用pip安装软件时会报

```
pip._vendor.distlib.DistlibException: Unable to locate finder for 'pip._vendor.distlib'
```

这时需要卸载掉pip setuptools，重新安装下

1. uninstall current pip:

   ```
   python -m pip uninstall pip setuptools
   ```

2. download `get-pip.py` from <https://bootstrap.pypa.io/get-pip.py>

3. execute get-pip script:

   ```
   python get-pip.py
   ```



#  



# 安装软件

```
Django (2.0.2)
celery (4.1.0)
mysqlclient (1.3.12) 连接mysql客户端
redis (2.10.6) 以redis作为后端broker
```

安装过程

```
(proj_env1) E:\virtualenv\nowamagic_venv\proj_env1\Scripts>pip install django
Collecting django
  Using cached Django-2.0.2-py3-none-any.whl
Collecting pytz (from django)
  Using cached pytz-2017.3-py2.py3-none-any.whl
Installing collected packages: pytz, django
Successfully installed django-2.0.2 pytz-2017.3

(proj_env1) E:\virtualenv\nowamagic_venv\proj_env1\Scripts>pip install celery
Collecting celery
  Using cached celery-4.1.0-py2.py3-none-any.whl
Requirement already satisfied: pytz>dev in e:\virtualenv\nowamagic_venv\proj_env1\lib\site-packages (from celery)
Collecting billiard<3.6.0,>=3.5.0.2 (from celery)
  Using cached billiard-3.5.0.3-py3-none-any.whl
Collecting kombu<5.0,>=4.0.2 (from celery)
  Using cached kombu-4.1.0-py2.py3-none-any.whl
Collecting amqp<3.0,>=2.1.4 (from kombu<5.0,>=4.0.2->celery)
  Using cached amqp-2.2.2-py2.py3-none-any.whl
Collecting vine>=1.1.3 (from amqp<3.0,>=2.1.4->kombu<5.0,>=4.0.2->celery)
  Using cached vine-1.1.4-py2.py3-none-any.whl
Installing collected packages: billiard, vine, amqp, kombu, celery
Successfully installed amqp-2.2.2 billiard-3.5.0.3 celery-4.1.0 kombu-4.1.0 vine-1.1.4

(proj_env1) E:\virtualenv\nowamagic_venv\proj_env1\Scripts>pip install mysqlclient
Collecting mysqlclient
  Using cached mysqlclient-1.3.12-cp36-cp36m-win_amd64.whl
Installing collected packages: mysqlclient
Successfully installed mysqlclient-1.3.12

(proj_env1) E:\virtualenv\nowamagic_venv\proj_env1\Scripts>pip install redis
Collecting redis
  Using cached redis-2.10.6-py2.py3-none-any.whl
Installing collected packages: redis
Successfully installed redis-2.10.6
```



#  



# 启动django项目

```
(proj_env1) e:\code\git\source\my>django-admin startproject django_celery_beat_test
```

现在结构如下

```
- django_celery_beat_test/
  - manage.py
  - django_celery_beat_test/
    - __init__.py
    - settings.py
    - urls.py
    - wsgi.py
```



# 定义 Celery 实例

**file:**  *django_celery_beat_test/django_celery_beat_test/celery.py*

```
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'django_celery_beat_test.settings')

app = Celery('django_celery_beat_test')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

然后你需要在你的django_celery_beat_test/django_celery_beat_test/__ init__.py模块中导入这个app。这可以确保在Django启动时加载app，以便@shared_task装饰器（稍后提及）使用它：

**file：**  *django_celery_beat_test/django_celery_beat_test/__ init__.py*

```
from __future__ import absolute_import, unicode_literals

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app

__all__ = ['celery_app']
```

其中

```
from __future__ import absolute_import
```

引入绝对导入是防止我们定义的celery.py模块与库模块celery冲突，具体作用可见[这里](http://blog.csdn.net/caiqiiqi/article/details/51050800)

```
app = Celery('django_celery_beat_test')
```

这是我们celery的实例，可以有很多实例，但django里一个可能就够了 

```
app.config_from_object('django.conf:settings', namespace='CELERY')
```

代表celery的配置文件在django的配置文件settings里，并且定义了命名空间CELERY，也即在settings里的celery的配置都得以CELERY开头，比如broker_url需要定义为CELERY_BROKER_URL

```
app.autodiscover_tasks()
```

使用上面的行，Celery会自动发现所有安装的应用程序中的任务，遵循tasks.py约定：

```
- app1/
    - tasks.py
    - models.py
- app2/
    - tasks.py
    - models.py
```

这样您就不必手动将各个模块添加到CELERY_IMPORTS设置中



## celery的配置

接着在settings.py中定义

```
CELERY_BROKER_URL = 'redis://:cds-china@172.20.3.3:6379/0'

#: Only add pickle to this list if your broker is secured
#: from unwanted access (see userguide/security.html)
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_RESULT_BACKEND = 'redis://:cds-china@172.20.3.3:6379/1'
```



## 创建app

```
python manage.py startapp demoapp
```

现在结构如下：

```
django_celery_beat_test/
├── demoapp
│   ├── __init__.py
│   ├── models.py
│   ├── tasks.py
│   ├── tests.py
│   ├── views.py
├── manage.py
├── django_celery_beat_test
│   ├── __init__.py
│   ├── celery.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
└── test.db
```

tasks.py内容如下：

```
# Create your tasks here
from __future__ import absolute_import, unicode_literals

from celery import shared_task


@shared_task
def add(x, y):
    return x + y


@shared_task
def mul(x, y):
    return x * y


@shared_task
def xsum(numbers):
    return sum(numbers)
```

这里使用shared_task，其与app.task区别见[这里](https://groups.google.com/forum/#!topic/celery-users/XiSDiNjBR6k)

在函数中调用task

views.py见下：

```
from __future__ import absolute_import, unicode_literals

import json

from django.http import HttpResponse
# Create your views here.
from .tasks import add


def index(request):
    print('1 + 1 = ?')
    r = add.delay(1, 1)
    print('r.get() = %s ' % r.get())
    resp = {'errorcode': 100, 'detail': 'Get success'}
    return HttpResponse(json.dumps(resp), content_type="application/json")
```

urls.py

```
from django.contrib import admin
from django.urls import path
from django.conf.urls import url
from demoapp.views import index

urlpatterns = [

    url(r'^$', index, name='index'),
    # url(r'^proj/', include('proj.foo.urls')),
    path('admin/', admin.site.urls),
]
```



# 启动celery worker

```
celery -A django_celery_beat_test worker --pool=solo -l info

(django_celery_beat_env) e:\code\git\source\my\django_celery_beat_test>celery -A
 django_celery_beat_test worker --pool=solo -l info

 -------------- celery@zq-PC v4.1.0 (latentcall)
---- **** -----
--- * ***  * -- Windows-7-6.1.7601-SP1 2018-02-08 13:19:43
-- * - **** ---
- ** ---------- [config]
- ** ---------- .> app:         django_celery_beat_test:0x3974198
- ** ---------- .> transport:   redis://:**@172.20.3.3:6379/0
- ** ---------- .> results:     redis://:**@172.20.3.3:6379/1
- *** --- * --- .> concurrency: 4 (solo)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . demoapp.tasks.add
  . demoapp.tasks.mul
  . demoapp.tasks.xsum
  . django_celery_beat_test.celery.debug_task
```

**注意这里celery4以上的版本在window上运行时，需要加上--pool=solo，否则在执行任务时会报**

```
ValueError: not enough values to unpack (expected 3, got 0)
```



# 启动server

```
python runserver 8000
```

浏览器访问http://127.0.0.1:8000/

看控制台会打印结果

```
1 + 1 = ?
[07/Feb/2018 18:37:59] "GET / HTTP/1.1" 200 43
r.get() = 2 
```

表示成功

 



# django_celery_beat

接下来看django_celery_beat模块

上面没有说celery beat，celery beat就是一个定时模块，并且包含crontab类似功能，后面是celery worker，可以说非常强大。

默认调度程序是celery.beat.PersistentScheduler，它只是跟踪本地[shelve](https://docs.python.org/dev/library/shelve.html#module-shelve)数据库文件中的最后一次运行时间。

但还有一哥们写的调度程序django_celery_beat，它以数据库做为载体，定时任务之类的记录在库里，并且有web admin界面控制。



## 安装

1.pip安装

```
pip install django-celery-beat
```

2.添加到INSTALLED_APPS

```
INSTALLED_APPS = (
    ...,
    'django_celery_beat',
)
```

3. migrate安装必要的库

```
python manage.py migrate


(django_celery_beat_env) e:\code\git\source\my\django_celery_beat_test>python ma
nage.py migrate
System check identified some issues:

WARNINGS:
?: (mysql.W002) MySQL Strict Mode is not set for database connection 'default'
        HINT: MySQL's Strict Mode fixes many data integrity problems in MySQL, s
uch as data truncation upon insertion, by escalating warnings into errors. It is
 strongly recommended you activate it. See: https://docs.djangoproject.com/en/2.
0/ref/databases/#mysql-sql-mode
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, django_celery_beat, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying django_celery_beat.0001_initial... OK
  Applying django_celery_beat.0002_auto_20161118_0346... OK
  Applying django_celery_beat.0003_auto_20161209_0049... OK
  Applying django_celery_beat.0004_auto_20170221_0000... OK
  Applying sessions.0001_initial... OK
```

\4. 启动celery beat的时候指定--scheduler

```
celery -A django_celery_beat_test beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```



## django admin

创建superuser

```
(django_celery_beat_env) e:\code\git\source\my\django_celery_beat_test>python ma
nage.py createsuperuser
Username (leave blank to use 'zq'): admin
Email address: zq@126.com
Password:
Password (again):
Superuser created successfully.
```

登录admin http://127.0.0.1:8000/admin/

如下：

![img](https://static.oschina.net/uploads/space/2018/0208/134004_2dyZ_914655.png)

可添加定时，任务等。