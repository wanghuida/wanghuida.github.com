---
layout: post
title: "MogileFS详解"
date: 2012-10-23 22:48
comments: true
sharing: true
footer: true
categories: [Mogilefs, Perl]
---

+ 经过一个月的实际应用，我想是时候应该总结一下mogilefs的使用了，如果想和作者交流参考[IRC教程](/blog/2012/10/13/irc-irssishi-yong-jiao-cheng/)

###MogileFS简介

> MogileFS是一个`开源`的`分布式文件系统`，配合cdn和squid，性能非常好(`15亿`文件没问题，其他数据不方便透露)

> 他由下面几个部分组成:

![mogilefs-summary](/images/post/mogilefs-summary.jpg "mogilefs-summary")

1. memcache：用来缓存查询结果，降低db压力
1. mogilefsd：就是tracker，用来接收请求并交给子进程(job)处理
1. mogstored：监控磁盘状态和文件的实际存储
1. client：支持perl，java，ruby，php，python
1. mysql：存储记录
1. util：日常维护管理的工具集
1. telnet：监控tracker，实时调整job的工具

<!-- more -->


###Mogstored

> 监控磁盘状态和文件的实际存储,启动mogstored会发现包含2个端口，默认的7501和7500

>> 7501用来监控磁盘状态

>> 7500用来处理实际的文件存储

![mogilefs-mogstored](/images/post/mogilefs-mogstored.jpg "mogilefs-mogstored")

####问题总结
+ 原本我一直以为mogstored自己实现了一套文件存储，但实际上不是。它使用的是WebDav，所以可以使用nginx代替默认的perlbal，实际用下来nginx更稳定
+ WebDav：一种基于 HTTP 1.1协议的通信协议.它扩展了HTTP 1.1，在GET、POST、HEAD等几个HTTP标准方法以外添加了一些新的方法，例如PUT(新增)、DELETE(删除)、MKCOL(创建目录)
+ 在mogstored的使用上，犯过一个错误：直接使用root启动mogstored，那么创建的文件夹和文件都是属于root，导致delete job无法删除该文件（同事陈磊发现）
+ 文件存储规则：/dev-xx/0/123/456/789/0123456789.fid，fid不足十位补0，通过fid进行切割变为目录，即分散又效率





###Mogilefsd

> 主进程负责接受请求，分配任务给子进程执行,下面详细介绍一下他的子进程


1. query：处理主进程分配的请求，包括util和tool的指令 
1. delete：根据删除队列对文件进行删除操作
1. replicate：复制文件直到满足mindevcount(文件复制几份)
1. fsck：检查磁盘文件和数据库是否匹配，不匹配时进行补救
1. monitor：监控子进程的状态，实时调整子进程个数
1. reaper：监控dead的磁盘，及时补救，把文件加到replicate queue
1. jobmaster：读取delete,replicate,rebalance,fsck队列，告诉主进程，主进程再交给对应的子进程处理

####问题总结    
+ query新建文件时，用剩余容量作权重，随机选择。所以新加入一块设备时，极有可能将成为热点设备，最好是一组设备一起加
+ 如果设备出现问题，直接格式化后直接使用会导致数据库记录还在，文件丢失，最终fsck和replicate队列堆积，正确的做法是把设备状态设置为dead，然后格式化用一个新的devid使用
+ 如果确定磁盘损坏，就一定要设置dead状态，reaper会自动补救





###Memcache

> 用来缓存查询结果，降低db压力，query内部存放`key对应的fid`和`fid对应的设备`

####问题总结
+ db压力下不来的原因是客户端取得文件时没有设置noverify参数，如果确定要使用memcache就一定要设置noverify，否则不使用memcache





###Mysql

> 用来存储记录，当然也可以使用其他数据库

1. host表：存储主机信息
1. device表：存储设备，一般单个device对应一块磁盘，也可以不这样
1. domain表：定义域，单个域下key唯一
1. class表：文件存放策略，包括份数，复制策略，校验和
1. server_settings表：部分配置信息和job配置信息
1. file表：文件信息
1. file_on表：文件存放在哪些存储设备
1. file_to_delete2表：删除队列表，升级后以前的不用了
1. file_to_queue表：type=1是fsck队列，type=2是rebalance队列
1. fsck_log表：fsck日志表，TYPE很多很诡异，参考[fsk探究](/blog/2012/09/29/mogilefsde-fscktan-jiu/)
1. file_to_replicate表：复制队列

####问题总结

+ 怀疑file_on表会有慢查询，其实不会太慢，因为Innodb默认asc排序

```
#file_on的索引情况
PRIMARY KEY (`fid`,`devid`),KEY `devid` (`devid`)
#下面的语句是不会慢的
select * from file_on where devid = ? and fid > ? order by fid asc limit 100
```

+ 怀疑多个replicate队列同时操作同一个fid时是否会出现bug，其实不会，因为replicate使用mysql的get_lock(fid),release_lock(fid)




###Utilities

1. mogadm：该工具直接管理mogilefs的主机、设备、域、类、从数据库、配置、fsck、rebalance
1. mogstat：观察fsck,rebalance,replication,delete运行状况
1. mogfiledebug：查看文件存储信息，也可以查file_on表
1. mogfetch：导出指定的文件
1. mogdelete：删除指定的文件
1. mogrename：更改文件的key
1. mogupload：插入一个文件
1. moglistfids，moglistkeys：批量列出记录，自己写脚本时可以使用





###Telnet

1. !version：显示mogilefs版本
1. !recent：显示当前query执行时间（可做监控）
1. !queue：显示等待执行的查询（可做监控）
1. !stats：从tracker启动开始就记录的一些累加信息和实时信息，比如处理请求的总数、job队列当前记录数、当前等待执行的请求的个数等
1. !watch：显示`警告`和`错误`信息（可做监控）
1. !jobs：显示当前各个job的个数
1. !want：`动态调整`job个数
1. !to：不常用，向子进程发送消息，用来调试的
1. !shutdown：关闭所有mogilefsd进程

