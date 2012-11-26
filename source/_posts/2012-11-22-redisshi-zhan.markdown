---
layout: post
title: "redis实战"
date: 2012-11-22 17:17
comments: true
sharing: true
footer: true
categories: [NoSQL, Share]
---

### redis是什么？

+ redis是一个开源的，先进的key-value存储服务器端
+ value可以包含多种结构，比如strings, hashes, lists, sets ,sorted sets
+ 官方网址：[http://redis.io](http://redis.io)
+ [freenode](/blog/2012/10/13/irc-irssishi-yong-jiao-cheng/)的聊天室名称是#redis

### 安装redis

+ mac下安装redis非常简单

```bash
$ brew install redis
```

+ linux下的安装步骤如下

```bash
#下载，解压和编译
$ wget http://redis.googlecode.com/files/redis-2.6.5.tar.gz
$ tar xzf redis-2.6.5.tar.gz
$ cd redis-2.6.5
$ make
```

###配置redis

+ redis的默认配置文件是redis.conf，里面有详细的注释
+ 下面是简单的单台redis配置

```
#base
daemonize yes                       #守护进程
port 6379                           #监听端口
bind 127.0.0.1                      #绑定的ip地址
timeout 0                           #断开空闲的客户端时间
loglevel notice                     #记录到错误日志的等级
logfile /usr/local/log/redis.log    #记录的日志文件
databases 16                        #可以选择的db个数

#rdb默认开启的持久化模式
save 900 1                          #save <秒> <改变个数>             
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes     #当bgsave错误时停止rdb写入
rdbcompression yes                  #采用LZF压缩
rdbchecksum yes                     #校验和
dbfilename dump.rdb                 #保存的文件名
dir /usr/local/var/db/redis/        #保存的路径
```

###启动redis server端

```bash
$ redis-server [/path/to/redis.conf]         #mac
$ src/redis-server [/path/to/redis.conf]     #linux
```

###打开client端，并测试

```bash
$ redis-cli             #mac
$ src/redis-cli         #linux

redis> set name wanghuida 
OK
redis> get name
"wanghuida"
```


