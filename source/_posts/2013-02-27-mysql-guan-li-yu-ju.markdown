---
layout: post
title: "Mysql 管理语句"
date: 2013-02-27 14:53
comments: true
sharing: true
footer: true
categories: [Mysql]
---

### 创建用户，设置密码，赋予权限

```
mysql > grant all privileges on mmd.* to 'mmd'@'localhost' identified  by '123456';
mysql > flush privileges;
mysql > drop user


# mysql 5.5+默认本地连接，请修改bind_address为0.0.0.0才能支持网络连接
mysql > grant select,update,insert,delete on mmd.* to 'mmd'@'%' identified  by '123456';
mysql > flush privileges;
mysql > show grants for 'mmd'@'%';

mysql > set password for 'mmd'@'%' = password('654321');
```

<!-- more -->

### create 数据库，数据表

```
mysql > create table staff ( 
      > id int not null auto_increment primary key, 
      > username varchar(50) not null, 
      > `desc` text 
      > ) engine=innodb default character set=utf8;

mysql > insert into staff (id,username,`desc`) values (NULL,'王惠达','测试数据');

mysql > select * from staff;
```

### 修改表结构

```
msyql > alter table staff modify username varchar(100) not null;

mysql > alter table staff add creation timestamp; 

mysql > alter table staff drop creation;
```

### 创建，删除 index

```
mysql > create index all_field on staff (id,username);

mysql > create index one_field on staff (username);

mysql > drop index one_field on staff;

```

### explain

```
mysql > explain select * from staff where username like '%惠达%'

+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | staff | ALL  | NULL          | NULL | NULL    | NULL |    1 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

mysql > explain select * from staff where id = 1;

+----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys     | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | staff | const | PRIMARY,all_field | PRIMARY | 4       | const |    1 |       |
+----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)

```

