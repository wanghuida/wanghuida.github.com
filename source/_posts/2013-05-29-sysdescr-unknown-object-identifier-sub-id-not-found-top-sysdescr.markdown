---
layout: post
title: "解决 sysDescr: Unknown Object Identifier (Sub-id not found: (top)  sysDescr)"
date: 2013-05-29 11:58
comments: true
sharing: true
footer: true
categories: [Share]
---


+ ubuntu安装snmp很方便

```
sudo apt-get install snmp snmpd
```

+ 配置我就跳过了，直接测试，问题来了

```
snmpwalk -v 2c -c public 127.0.0.1 sysDescr

# 显示 sysDescr: Unknown Object Identifier (Sub-id not found: (top) -> sysDescr)
```

+ 网上搜一下，没有正确可行的办法，但是也是有收获

1. 直接使用snmpwalk -v 2c -c public 127.0.0.1 是没有问题的
2. 那就说明问题出在mibs上
3. 尝试解决一下

```
sudo apt-get install snmp-mibs-downloader

vim /etc/snmp/snmp.conf
#全部注释下～～～
#再测试就没问题了，使用了新的mibs
snmpwalk -v 2c -c public 127.0.0.1 sysDescr

```
