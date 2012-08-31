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



