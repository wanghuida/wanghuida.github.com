---
layout: post
title: "MogileFS安装配置"
date: 2012-11-22 17:19
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---

###基本介绍
+ mogilefs是一个分布式的文件系统解决方案
+ 特点是:无单点失效的问题，自动复制无需raid，easy的命名空间，自修复机制，成熟的管理工具，源码清晰推荐阅读

<!-- more -->

###安装基本组件

```bash
yum groupinstall "Development Tools"
#cpan是perl的包管理工具
yum install cpan
```

###安装mogilefs

```bash
#进入cpan,初始化直接自动就好
cpan
> install IO::AIO
> install MogileFS::Client
> install MogileFS::Utils
#如果有问题直接用force install
> install MogileFS::Server 
```

###安装配置mysql

```bash
#安装mysql
yum install mysql mysql-server
mysqladmin -u root password 123456
#进入mysql,新建mogile用户
mysql -uroot -p123456
mysql> create database mogilefs;
mysql> grant all on mogilefs.* to mogile@localhost identified by '123456';
mysql> flush privileges;

#导入mogilefs表
mogdbsetup --dbpass=123456 --yes
```

###mogilefs配置

+ mogilefsd配置

```bash
#/etc/mogilefs是默认的配置路径，-c可以修改
mkdir -p /etc/mogilefs
cd /etc/mogilefs
vim mogilefsd.conf

#内容如下
pidfile = /tmp/mogilefsd.pid    #存放pid
daemonize = 1                   #守护进程

db_dsn  = DBI:mysql:mogilefs:127.0.0.1  #数据库配置
db_user = mogile 
db_pass = 123456 

conf_port = 7001                #监听端口
node_timeout = 2                #节点超时时间 
rebalance_ignore_missing = 1    #rebalance时会删除文件，删除时如果找不到文件不报错直接跳过 

query_jobs = 20                 #正真处理请求的job个数
delete_jobs = 3                 #处理删除队列的job个数
replicate_jobs = 5              #处理复制队列的job个数
reaper_jobs = 1                 #设备设置为dead后，处理补救工作的job个数 
                                #其实还有其他job，默认为1个
```

+ mogstored配置

```bash
#设置文件存储路径，不要忘记修改拥有者
mkdir -p /var/mogdata
adduser --gid daemon mogile
chown mogile.daemon /var/mogdata
vim mogstored.conf

#内容如下
httplisten = 0.0.0.0:7500       #文件存储端口
mgmtlisten = 0.0.0.0:7501       #监控磁盘状态端口
docroot = /var/mogdata          #文件存储路径

pidfile = /tmp/mogstored.pid    #存储pid
daemonize = 1                   #守护进程
```

###启动mogilefs

```bash
vim /etc/security/limits.conf
#内容如下，让mogile可以打开更多的连接数
mogile soft nofile 65535
mogile hard nofile 65535

#切换用户启动
su mogile -c 'mogstored'
su mogile -c 'mogilefsd'

#验证已经启动成功
ps auxf | grep mogile
mogadm check

```

###已经安装好了，接来应该做点什么呢

+ 学习mogadm指令，因为mogadm是必会工具
+ 试着client连接mogilefsd存储文件吧
