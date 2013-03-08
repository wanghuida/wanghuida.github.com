---
layout: post
title: "Mysql 日志分类"
date: 2013-02-27 14:52
comments: true
sharing: true
footer: true
categories: [Mysql]
---




### general-log : 记录建立的客户端连接和执行的语句。

```
# my.cnf 一般不会打开，耗io
general_log_file        = /var/log/mysql/mysql.log

# 当你怀疑在客户端发生了错误并想确切地知道该客户端发送给mysqld的语句时，该日志可能非常有用。
# 一般都是临时打开
mysql > show variables like '%general_log%';  

+------------------+------------------------------+
| Variable_name    | Value                        |
+------------------+------------------------------+
| general_log      | OFF                          |
| general_log_file | /var/lib/mysql/idc08-002.log |
+------------------+------------------------------+
2 rows in set (0.00 sec)

mysql > SET GLOBAL general_log_file = '/tmp/mysql.log';
mysql > SET GLOBAL general_log = 'ON';

# 已经可以监控请求语句了，用完要记得关闭
```

<!-- more -->

### log-bin : 记录所有更改数据的语句。还用于复制。

+ 接替日志: flush logs语句或执行mysqladmin flush-logs
+ 运行服务器时若启用二进制日志则性能大约慢1%。但是，二进制日志的好处，即用于恢复并允许设置复制超过了这个小小的性能损失。
+ mysqld在每个二进制日志名后面添加一个数字扩展名。每次你启动服务器或刷新日志时该数字则增加。如果当前的日志大小达到max_binlog_size，还会自动创建新的二进制日志。如果你正使用大的事务，二进制日志还会超过max_binlog_size：事务全写入一个二进制日志中，绝对不要写入不同的二进制日志中。

```
log_bin             = /var/log/mysql/mysql-bin.log
max_binlog_size     = 100M
expire_logs_days    = 10
#binlog_do_db       = db1,db2,db3
#binlog_ignore_db   = include_database_name
```

### slow-log : 记录所有执行时间超过long_query_time秒的所有查询或不使用索引的查询。

+ 5.1以上的版本已经支持毫秒级

```
log_slow_queries = /var/log/mysql/mysql-slow.log
long_query_time = 0.5 
```

### error-log : 记录启动、运行或停止mysqld时出现的问题。

```
# my.cnf
log_error = /var/log/mysql/error.log
```


