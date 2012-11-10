---
layout: post
title: "安装配置hive"
date: 2012-11-10 14:59
comments: true
sharing: true
footer: true
categories: [Hadoop, Java]
---

+ 先安装hadoop请参考[这篇文章](/blog/2012/11/02/an-zhuang-pei-zhi-hadoop/)
+ hive介绍：通过类似于sql语句的方式去计算结果，数据都存放在hdfs

###下载并解压hive

+ 给个官方[链接](http://www.apache.org/dyn/closer.cgi/hive/)

```
tar -xzvf hive-x.y.z.tar.gz
cd hive-x.y.z
PATH=$PATH:/Users/william/project/hadoop-1.0.4/bin:/Users/william/project/hive-0.8.1/bin
. ~/.bash_profile
```

+ 直接在终端输入hive，然后输入show tables;然会OK，就行了

<!-- more -->

###创建测试表

```
create table test (id INT,name STRING);
create table test2 (id INT,name STRING) partitioned by (ds STRING);
create table result (id int,name string);
```

###导入数据

```
load data local inpath '/Users/william/project/hive-0.8.1/examples/files/kv1.txt' overwrite into table test;
load data local inpath '/Users/william/project/hive-0.8.1/examples/files/kv2.txt' overwrite into table test2 partition (ds='2012-10-01');
load data local inpath '/Users/william/project/hive-0.8.1/examples/files/kv3.txt' overwrite into table test2 partition (ds='2012-10-02');
```

###操作数据

```
select count(*) from test2 where ds='2012-10-01';
insert overwrite directory '/tmp/hdfs_out' select t.* from test2 t where t.ds='2012-10-01';
insert overwrite local directory '/tmp/local_out' select a.* from test a;
insert overwrite table result select a.* from test a where a.id < 100;
```

###换种风格操作数据

```
from test t                                                                 
    insert overwrite directory '/tmp/other_out' select t.* where t.id < 60      
    insert overwrite local directory '/tmp/other_out2' select t.* where t.id > 60;
```

+ 还可以自己写脚本做数据转换器
