---
layout: post
title: "常用命令"
date: 2012-08-30 13:14
comments: true
sharing: true
footer: true
categories: [Share]
---

###过滤记录

+ -l自动chomp，结果加$/
+ -a自动分割

```bash
tail -f access-2012-08-30.log | grep -v '*' \ 
    | perl -w -lane '$a = $F[@F-1];print if $a > 50000 '

```

###获取URL内容
+ -I 获取头信息
+ -x 通过哪台服务器
+ -A 设置user agent,可以让log过滤更方便
```
curl -I -x 127.0.0.1  "http://test/abc.jpg" -A "huida"
```

###批量删除一天以前的文件
```
find /dev/shm/ -type f -mtime +1 | xargs rm
```

###计算tcp连接数
```
su -
netstat -lant | wc -l
```
