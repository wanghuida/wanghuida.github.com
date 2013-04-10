---
layout: post
title: "克隆 slave"
date: 2013-04-07 17:54
comments: true
sharing: true
footer: true
categories: [Mysql]
---

### 已经有一台master 和 一台slave1，从slave1上克隆一个slave2

+ 关闭slave1 的 io_thread，那么slave1就不会读取master的bin-log了，也不会写自己的relay-log了

```
mysql> stop slave io_thread
```

+ 查看slave1 的 processlist，确保sql_thread在等待io_thread

```
mysql> show processlist\G;

# 显示如下结果，可以继续操作
State: Slave has read all relay log; waiting for the slave I/O thread to update it
```

<!-- more -->

+ 停止slave1的slave, 导出数据，并复制数据到slave2【如果数据大可以压缩】

```
mysql> stop slave

$ mysqldump -uroot -p mmd > mmd.sql

$ scp mmd.sql wanghuida@192.168.100.100:mmd.sql
```

+ slave2 创建数据库，并导入数据

```
$ mysqladmin -uroot -p create mmd
$ mysql -uroot -p mmd < mmd.sql
```

+ 修改 slave2 配置， 并重启

```
$ vim /etc/mysql/my.cnf

server-id       = 3
log_bin         = /var/log/mysql/mysql-bin.log
expire_logs_days    = 10
max_binlog_size     = 100M
replicate-do-db     = mmd
read_only           = 1

$ /etc/init.d/mysqld restart

```

+ 查看slave1上的从状态

```
mysql> show slave status\G;
```

+ 设置slave2 的主从同步配置，再启动就可以了

```
mysql> change master to 
    master_host = '192.168.100.153', 
    master_user = 'backup', 
    master_password = '123456', 
    master_log_file = 'mysql-bin.000016', 
    master_log_pos = 115910;

mysql> start slave
```
