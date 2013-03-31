---
layout: post
title: "ubuntu 修复locale错误"
date: 2013-03-27 10:55
comments: true
sharing: true
footer: true
categories: [Share]
---

+ locale 配置错误：Cannot set LC_ALL to default locale: No such file or directory

<!-- more -->

```
root@server:~# locale
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=en_US.UTF-8
LANGUAGE=en_US:en
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC=en_US
LC_TIME=en_US
LC_COLLATE="en_US.UTF-8"
LC_MONETARY=en_US
LC_MESSAGES="en_US.UTF-8"
LC_PAPER=en_US
LC_NAME=en_US
LC_ADDRESS=en_US
LC_TELEPHONE=en_US
LC_MEASUREMENT=en_US
LC_IDENTIFICATION=en_US
LC_ALL=
```

+ 修复：重新生成配置

```
sudo locale-gen en_US en_US.UTF-8
dpkg-reconfigure locales 
```
