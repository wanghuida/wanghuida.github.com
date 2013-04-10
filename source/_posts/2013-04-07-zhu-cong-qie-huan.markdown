---
layout: post
title: "主从切换"
date: 2013-04-07 20:32
comments: true
sharing: true
footer: true
categories: [Mysql]
---

### 1个master，1个slave1，1个slave2，关闭master，让slave1做master，slave2同步新master

+ slave1开启同步账号

```
mysql > grant replication slave on *.* to 'backup'@'192.168.0.152' identified by '123456';
mysql > grant replication slave on *.* to 'backup'@'192.168.0.153' identified by '123456';
mysql > flush privileges;
```

+ 锁住master的写

```
mysql > flush tables with read lock;

# 关闭锁的命令：unlock tables;
```

<!-- more -->

+ 查看slave1和slave2的同步状态

```
mysql > show processlist;

# 确保 Slave has read all relay log; waiting for the slave I/O thread to update it
```

+ 修改slave1的配置，并重启

```
bind-address = 0.0.0.0

$ sudo service mysql restart
```

+ 停止slave1的slave，设置为master

```
mysql > stop slave;
mysql > reset master;
```

+ 停止slave2的slave，重新设置master

```
mysql > stop slave;

mysql > change master to
      > master_host = '192.168.0.151',
      > master_user = 'backup',
      > master_password = '123456';

mysql> start slave;
```

+ 测试是否成功
```
mysql > create table `test` ( id int not null auto_increment, `name` varchar(100) not null, primary key (`id`) );
mysql > drop table test;
```
