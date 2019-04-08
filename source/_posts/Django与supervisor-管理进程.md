---
title: Django与supervisor 管理进程
tags:
- Django
- Linux
---



## 1、前言

在Django项目中，我们需要用到一些独立于Django框架外的脚本。这样一些脚本可能需要独立的持续运行，且具有很强的可维护性，这个时候supervisor就可以排上用场了。

![](https://ws1.sinaimg.cn/large/d126accegy1fyq9bijxgvj206h08hjrr.jpg)

<!--more-->

- 基于python编写，安装方便

- 进程管理工具，可以很方便的对用户定义的进程进行启动，关闭，重启，并且对意外关闭的进程进行重启 ，只需要简单的配置一下即可，且有web端，状态、日志查看清晰明了。

- 组成部分 

  - supervisord[服务端，所以要通过这个来启动它]
  - supervisorctl[客户端，可以来执行stop等命令]

- 官方文档地址：

  http://supervisord.org/

## 2、安装

直接使用pip进行

```
pip install supervisor
```

## 3、使用说明

使用supervisor很简单，只需要修改一些配置文件，就可以使用了。

#### 3.1 查看默认配置

运行

```
echo_supervisord_conf
```

即可看到默认配置情况，但是一般情况下，我们都不要去修改默认的配置，而是将默认配置重定向到另外的文件中，不同的进程运用不同的配置文件去对默认文件进行复写即可。

```
echo_supervisord_conf > /etc/supervisord.conf
```

默认配置说明

> 默认的配置文件是下面这样的，但是这里有个坑需要注意，supervisord.pid 以及 supervisor.sock 是放在 /tmp 目录下，但是 /tmp 目录是存放临时文件，里面的文件是会被 Linux 系统删除的，一旦这些文件丢失，就无法再通过 supervisorctl 来执行 restart 和 stop 命令了，将只会得到 unix:///tmp/supervisor.sock 不存在的错误 。

```ini
[unix_http_server]
;file=/tmp/supervisor.sock   ; (the path to the socket file)
;建议修改为 /var/run 目录，避免被系统删除
file=/var/run/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for ;all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))
...

[supervisord]
;logfile=/tmp/supervisord.log ; 日志文件(main log file;default $CWD/supervisord.log)
;建议修改为 /var/log 目录，避免被系统删除
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; 日志文件大小(max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; 日志文件保留备份数量(num of main logfile rotation backups;default 10)
loglevel=info                ; 日志级别(log level;default info; others: debug,warn,trace)
;pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
;建议修改为 /var/run 目录，避免被系统删除
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
;设置启动supervisord的用户，一般情况下不要轻易用root用户来启动，除非你真的确定要这么做
;user=chrism                 ; (default is current user, required if root)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)
;umask=022                   ; (process file creation umask;default 022)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY="value"     ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

[unix_http_server]
file=/tmp/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisorctl]
; 必须和'unix_http_server'里面的设定匹配
;serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
;建议修改为 /var/run 目录，避免被系统删除
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set

。。。

[include]
files = /etc/supervisor/*.conf
```

配置文件都有说明，且很简单，就不做多的描述了，在上面有一些建议修改的目录，若做了修改，则应先创建这些文件，需要注意权限问题，很多错误都是没有权限造成的。

#### 3.2 启动服务端

现在，让我们来启动supervisor服务。

``````
supervisord -c /etc/supervisord.conf
``````

查看supervisord 是否运行：

``````
ps aux|grep superviosrd
``````

```
output:xxxx   82039      1  0 11:22 ?        00:00:00 /usr/local/bin/python /usr/local/bin/supervisord -c /etc/supervisord.conf
```

#### 3.2 项目配置及运行

上面我们已经把 supervisrod 运行起来了，现在可以添加我们要管理的进程的配置文件。可以把所有配置项都写到 supervisord.conf 文件里，但并不推荐这样做，而是通过 include 的方式把不同的程序（组）写到不同的配置文件里，对，就是默认配置中的最后的那个include。下面来对项目进行简单的配置。
假设我们把项目配置文件放在这个目录中:`/etc/supervisor/`
则我们需要修改`/etc/supervisord.conf` 中的include为：

``````ini
 [include]
 files = /etc/supervisor/*.conf
``````

测试py文件：

``````python
import time
while True:
    print('i am ok')
    time.sleep(5)
    
``````

以下为配置文件目录`/etc/supervisor/test.conf`：

```ini
[program:testpy]
command=python test.py              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
directory=/root/myporject/               ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
priority=999                  ; the relative start priority (default 999)
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
stdout_logfile = /root/myporject/supervisor.log
```

配置完成以后，即可运行：

``````
supervisord -c /etc/supervisord.conf
``````

查看运行状态

``````
[root@localhost ~]# supervisorctl status
testpy                           RUNNING   pid 2391, uptime 0:00:54

``````

打开浏览器，输入127.0.0.9001,输入用户名与密码（如果配置文件中inet_http_server中作了设置），可以看到下面这个界面：

![](https://ws1.sinaimg.cn/large/d126accegy1fyqajnbn72j20me066t8x.jpg)

#### 3.3 使用supervisorctl

在启动服务之后，运行：

``````
supervisorctl -c /etc/supervisord.conf
``````

或者直接`supervisorctl`

```
[root@localhost ~]# supervisorctl
testpy                           RUNNING   pid 2391, uptime 0:01:45
```

若成功，则会进入supervisorctl的shell界面，有以下方法：

- status    			   # 查看程序状态
- stop testpy               # 关闭 testpy 程序
- start testpy               # 启动 testpy 程序
- restart testpy           # 重启 testpy 程序
- reread                      ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
- update                     ＃ 重启配置文件修改过的程序

执行相关操作后，可以在web端看到具体的变化情况，如stop 程序

``````
stop testpy
``````



![](https://ws1.sinaimg.cn/large/d126accegy1fyqaemp03ej20lv061t8y.jpg)

其实，也可以不使用supervisorctl shell界面，而在bash终端运行：

```
$ supervisorctl status
$ supervisorctl stop usercenter
$ supervisorctl start usercenter
$ supervisorctl restart usercenter
$ supervisorctl reread
$ supervisorctl update 
```

#### 3.4 多个进程管理

按照官方文档的定义，一个 [program:x] 实际上是表示一组相同特征或同类的进程组，也就是说一个 [program:x] 可以启动多个进程。这组进程的成员是通过 numprocs 和 process_name 这两个参数来确定的，这句话什么意思呢，我们来看这个例子。

```
; 设置进程的名称，使用 supervisorctl 来管理进程时需要使用该进程名
[program:foo] 

; 可以在 command 这里用 python 表达式传递不同的参数给每个进程
command=python server.py --port=90%(process_num)02d

directory=/home/python/tornado_server ; 执行 command 之前，先切换到工作目录

; 若 numprocs 不为1，process_name 的表达式中一定要包含 process_num 来区分不同的进程
numprocs=2                   
process_name=%(program_name)s_%(process_num)02d; 

user=oxygen                 ; 使用 oxygen 用户来启动该进程

autorestart=true  ; 程序崩溃时自动重启

redirect_stderr=true      ; 重定向输出的日志

stdout_logfile = /var/log/supervisord/
tornado_server.log
loglevel=info
```

上面这个例子会启动两个进程，process_name 分别为 foo:foo_01 和 foo:foo_02。通过这样一种方式，就可以用一个 [program:x] 配置项，来启动一组非常类似的进程。
 更详细配置，点击[这里](https://link.jianshu.com?t=http://supervisord.org/configuration.html#program-x-section-settings)

Supervisor 同时还提供了另外一种进程组的管理方式，通过这种方式，可以使用 supervisorctl 命令来管理一组进程。跟 [program:x] 的进程组不同的是，这里的进程是一个个的 [program:x] 。

```
[group:thegroupname]
programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
priority=999                  ; the relative start priority (default 999)
```

当添加了上述配置后，progname1 和 progname2 的进程名就会变成 thegroupname:progname1 和 thegroupname:progname2 以后就要用这个名字来管理进程了，而不是之前的 progname1。

以后执行 supervisorctl stop thegroupname: 就能同时结束 progname1 和 progname2，执行 supervisorctl stop thegroupname:progname1 就能结束 progname1。

## 4. 结尾

实际上，默认情况下，supervisored 也是一个进程，最理想的的情况应该是将其安装为系统服务，安装方法可以参考[这里](https://link.jianshu.com?t=http://serverfault.com/questions/96499/how-to-automatically-start-supervisord-on-linux-ubuntu),安装脚本参考[这里](https://link.jianshu.com?t=https://github.com/Supervisor/initscripts),由于没有做具体的实验，此处不展开说明。

其实还有一个简单的方法，因为 Linux 在启动的时候会执行 /etc/rc.local 里面的脚本，所以只要在这里添加执行命令就可以

```
# 如果是 Ubuntu 添加以下内容
/usr/local/bin/supervisord -c /etc/supervisord.conf

# 如果是 Centos 添加以下内容
/usr/bin/supervisord -c /etc/supervisord.conf
```

以上内容需要添加在 exit 命令前，而且由于在执行 rc.local 脚本时，PATH 环境变量未全部初始化，因此命令需要使用绝对路径。

在添加前，先在终端测试一下命令是否能正常执行，如果找不到 supervisord，可以用如下命令找到

```
sudo find / -name supervisord

output:
/usr/local/bin/supervisord
```

## 参考资料：

> 这段日子真的很难	https://www.jianshu.com/p/bf2b3f4dec73