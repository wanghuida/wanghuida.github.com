---
layout: post
title: "nginx反向代理自动urldecode问题"
date: 2012-11-07 16:40
comments: true
sharing: true
footer: true
categories: [Share]
---

###今天妖孽了，发现nginx反向代理会自动urldecode

+ 同事监控两个nginx的access_log，惊人的发现日志里的url不同
+ 去掉反斜杠就ok了，不太清楚nginx内部是如何处理的


```
location / {
    proxy_pass        http://localhost:8000/;
}
#修改为下面这样后，一切OK了
location / {
    proxy_pass        http://localhost:8000;
}
```
