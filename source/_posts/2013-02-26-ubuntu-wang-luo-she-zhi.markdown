---
layout: post
title: "ubuntu 网络设置"
date: 2013-02-26 15:22
comments: true
sharing: true
footer: true
categories: [Share]
---



### ubuntu的网络设置涉及到如下文件：

```
/etc/network/interfaces # 网络接口配置，包括网络接口说明、IP地址、子网掩码、网关等
/etc/resolv.conf        # DNS服务器设置
/etc/hostname           # 主机名设置
/etc/hosts              # 域名解析映射
/etc/hosts.allow        # IP访问允许规则
/ect/hosts.deny         # IP访问禁止规则
```

+ 修改网络配置文件后，要重启网络接口，使用命令：/etc/init.d/networking restart
+ 重启单个网口可以使用ifdown eth0,ifup eth0, ifconfig

```
ifconfig eth1 192.168.0.204/24
ifconfig eth1 down/up
```

<!-- more -->


### dhcp

```
vim /etc/network/interfaces
auto eth0               # 设置eth0开机自动加载
iface eth0 inet dhcp    # 定义网络接口eth0为Internet，DHCP方式。
```

### static

```
auto eth0
iface eth inet static
address 192.168.0.110
gateway 192.168.0.1
netmask 255.255.255.0

dns-nameservers 8.8.8.8
```

### dns

```
# 最好不要改这个，重启后就没了
vim /etc/resolv.conf

nameserver x.x.x.x  # 首要DNS服务器
nameserver x.x.x.x  # 备用DNS服务器
```
