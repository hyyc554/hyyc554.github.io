---
title: Centos7环境 Mongodb 的安装及远程连接
date: 2019-05-05 18:42:08

tags:
  - 数据库
---

此流程仅本人测试，没有报错。折腾了一会，出了解决不了的BUG还是卸载重装比较方便。

<!--more-->

## 一、安装

① 把Mongo的安装配置添加的yum中

```bash
vi /etc/yum.repos.d/mongodb-org-4.0.repo
```

把下面配置复制到文件中

``````bash
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
``````

② 运行安装命令

```bash
sudo yum install -y mongodb-org
```

③ 设置数据储存路径
默认下mongo的储存路径是 /data/db ，如果此时系统中没有这个文件，是不会启动成功的。所以要手动穿件这个文件夹

``````bash
mkdir -p /data/db
``````

④ 启动Mongo

``````bash
sudo service mongod start
``````

⑤ 连接本地的Mongo

``````bash
mongo
``````

此时Mongo的安装已经完成，上面日志中有警告啥的可以通过配置解决，但不影响使用。

参考资料：https://docs.mongodb.com/master/mongo/

## 二、Mongo的远程连接

注意：Mongo的远程连接需要打开权限控制 
本教程是不过多涉及权限问题，权限详情可参考： 
http://www.cnblogs.com/hanyinglong/archive/2016/07/25/5704320.html

① 添加新的用户
首先添加个管理员账号（root权限）：

```javascript
 db.createUser({
    user:"root",
    pwd:"password",
    roles:[{role:"root",db:"admin"}]
    })
```


添加个普通账号（读写权限）： (需要先用root登陆)
（命令中的db 代表用户所分配的数据库）

``````javascript
db.createUser({
                　　user:"hyc",                                   
                　　pwd:"123456",
                　　roles:[{role:"readWrite",db:"test"}]
           　　});
``````


② 修改配置文件

``````
vi /etc/mongod.conf
``````

注释掉：

``````bash
# bindIp: 127.0.0.1  # Listen to local interface only, comment to listen on all interfaces.

``````

添加：

``````bash
security:
    authorization: enabled
``````

③ 重启Mongo 远程连接

``````
service mongod restart
``````

④ 开启端口访问

``````bash
firewall-cmd --zone=public --permanent --add-port=27017/tcp; firewall-cmd --reload
``````



打开ROBO（mongo 可视化工具）：

[![iXEZcD.png](https://s1.ax1x.com/2018/11/13/iXEZcD.png)](https://imgchr.com/i/iXEZcD)

![iXArOH.png](https://s1.ax1x.com/2018/11/13/iXArOH.png)




最后点击Save就可以愉快的使用啦
--------------------- 
