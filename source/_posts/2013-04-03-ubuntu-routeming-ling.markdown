---
layout: post
title: "ubuntu route命令"
date: 2013-04-03 18:55
comments: true
sharing: true
footer: true
categories: [Share]
---



### route是非常常用的命令，作用是增加一条静态路由

+ 以前觉得这条命令很难用，其实很简单～～

```
# route 直接打印静态路由表 -n 选项代表把域名解析成ip
$ route -n
```


### route 命令的基本格式

+ route 操作 目标地址 网关 设备 

```
# 例1: eth0发往192.168.0.0网段的数据，都交给192.168.0.1网关
# add就是操作，-net 192.168.0.0/24 就是目标网段
# gw 192.168.0.1 是网关，dev eth0就是设备
$ route add -net 192.168.0.0/24 gw 192.168.0.1 dev eth0

# 例2: eth0默认网段的数据，都交给192.168.0.1网关
$ route add default gw 192.168.0.1 dev eth0

# 例3: eth0发往192.168.0.105主机的数据，都交给192.168.0.1网关
$ route add -host 192.168.0.105 gw 192.168.0.1 dev eth0

```
