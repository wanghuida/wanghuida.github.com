---
layout: post
title: "Mysql 主从设置"
date: 2013-02-27 14:52
comments: true
sharing: true
footer: true
categories: [Mysql]
---

### 设置同步账号

```
mysql > grant replication slave on *.* to 'backup'@'%' identified by '123456';
mysql > flush privileges;
```

### master 配置

<!-- more -->

```
server-id       = 1
log_bin         = /var/log/mysql/mysql-bin.log
max_binlog_size     = 100M
expire_logs_days    = 10
binlog_do_db        = mmd

service mysql restart
```

### slave 配置

```
server-id           = 2 
log_bin             = /var/log/mysql/mysql-bin.log
expire_logs_days    = 10
max_binlog_size     = 100M
replicate-do-db     = mmd 
read_only           = 1

service mysql restart
```

### 同步现有数据

```
# 强制读锁
mysql > flush tables with read lock;
# 导出数据,如果是myisam表的话，加上--lock-all-tables参数，如果是innodb表的话，加上--single-transaction参数。如果数据量很大，建议停服务生成数据快照
$ mysqldump -uroot -p63137246 --single-transaction mmd > mmd.sql
# 记录同步点
mysql > show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000007 |      887 | mmd          |                  |
+------------------+----------+--------------+------------------+
# 关闭读锁
mysql > unlock tables;

$ scp mmd.sql wanghuida@192.168.30.114:mmd.sql
$ mysql -uroot -p***** mmd < mmd.sql
```

### slave 同步设置

```
mysql > change master to
      > master_host='192.168.30.113', 
      > master_user='backup', 
      > master_password='123456', 
      > master_log_file='mysql-bin.000007', 
      > master_log_pos=887;

mysql > start slave;
```

### 附加配置

+ 注释：为了保证事务InnoDB复制设置的最大可能的耐受性和一致性，应在主服务器的my.cnf文件中使用innodb_flush_log_at_trx_commit=1和sync-binlog=1。这样每次提交都会写入日志文件，性能有一些损失
