---
layout: post
title: "Mysql 状态检查"
date: 2013-02-27 14:52
comments: true
sharing: true
footer: true
categories: [Mysql]
---


### 最简单的status

+ 可以了解一下mysql基本的状态

```
$ mysql -u -p -h 
mysql > status
```

<!-- more -->

+ 另外可以添加 -i 5 参数，让其每五秒自动刷新一次。 

```
mysqladmin -uroot -p密码 status -i 5 
```

### 查看mysql详细的状态信息

+ 里面包含mysql所有的状态信息，分散在各处，很难理解，下会再推荐一个更有效的工具

```
mysql > show status; (session 级别的)
mysql > show global status; (server 级别的, 一般都使用这个)
```

### mysqlreport

+ 官方网站: http://hackmysql.com/mysqlreport 
+ 该工具使用方便，数据也很清晰，非常推荐使用该工具

```
./mysqlreport --host=10.0.1.111 --user=**** --password=****
```

### 更快速的查看mysql 表缓存

```
mysql > show open tables;
```

### show processlist

+ 当感觉到每个线程有问题时，可以使用这条命令看下情况

+ 案例：DBA通知我processlist里有很多空链接，我发现mysql链接都是持久化的。。并且没有关闭


### show (session | global) variables

+ 查看当前服务器配置变量

```
# 很多情况下都不希望重启mysql服务，所以可以动态调整变量
# 例如输出log
set global log=/tmp/query.log

# 希望重启后也会应用到，可以添加到my.cnf
```

### show table status [from db] [like '%table%']

+ 显示具体某个表的状态信息


