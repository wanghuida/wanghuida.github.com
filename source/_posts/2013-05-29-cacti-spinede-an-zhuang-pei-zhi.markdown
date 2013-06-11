---
layout: post
title: "cacti spine的安装配置"
date: 2013-05-29 15:35
comments: true
sharing: true
footer: true
categories: [Share]
---


+ cacti官方有详细的配置流程，这里只是翻译和错误的解决


```bash
wget http://www.cacti.net/downloads/spine/cacti-spine-0.8.8a.tar.gz
tar -zxvf cacti-spine-0.8.8a.tar.gz
cd cacti-spine-0.8.8a/

./configure
# configure: error: Cannot find MySQL headers.  Use --with-mysql= to specify non-default path.
# 下载安装mysqlclient开发包
sudo apt-get install libmysqlclient-dev

./configure
# configure: error: Cannot find SNMP headers.  Use --with-snmp= to specify non-default path.
# 下载安装snmp开发包
sudo apt-cache search snmp | grep dev
sudo apt-get install libsnmp-dev

./configure
make
make install
```

+ 搞定配置文件

```
cp /usr/local/spine/etc/spine.conf.dist /usr/local/spine/etc/spine.conf
```

+ admin登录cacti页面, 选择Settings的Paths选项卡，输入完整的spine路径[Alternate Poller Path]

+ 点击Settings的Poller选项卡，从Poller Type里选择spine
