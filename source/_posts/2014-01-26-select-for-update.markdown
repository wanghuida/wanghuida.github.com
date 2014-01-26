---
layout: post
title: "select for update"
date: 2014-01-26 15:37
comments: true
sharing: true
footer: true
categories: [Mysql]
---


#### 我的使用场景

+ 有一大堆产品需要在网上出售，每个产品都有数量，表结构如下

```
id      product_name      count
1       100元充值卡       5
2       iphone5s          2
```

+ 每次出售一个count数量就减1，如何保证并发操作的count准确性

```
# 伪代码

transaction start

try 
    // 并发执行会锁行，这样就确保了准确性
    row = DB::select('select * from product where id=? for update', product_id)
    row.count = row.count - 1
    row.save()
    commit
catch
    rollback

```

+ 需要注意的问题是mysql需要innodb表，并且sql要吃上索引，否则锁表
