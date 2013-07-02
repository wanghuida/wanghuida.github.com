---
layout: post
title: "php图片处理ubuntu配置"
date: 2013-07-02 10:08
comments: true
sharing: true
footer: true
categories: [Php]
---


+ 安装imagemagick 和图片库

```
sudo apt-get install libpng12-0 libpng12-dev libpng3
sudo apt-get install libtiff4 libtiff4-dev libtiffxx0c2
sudo apt-get install libjasper-dev libjpeg-dev
sudo apt-get install imagemagick

# 显示 imagemagick 支持的图片类型 
convert -list format
convert -list configure
```

<!-- more -->

+ 安装 php和php模块

```
sudo apt-get install php5 php5-curl php5-gd php5-mysql php5-fpm php5-imagick php5-xcache php5-cli php5-common php5-mcrypt
```

+ 配置xcache

```
xcache.size  =  64M
xcache.count =  8
xcache.slots =  16K
```


+ 配置fpm里的php.ini

```
error_log = /tmp/php-error.log
post_max_size = 20M
upload_max_filesize = 20M
date.timezone = PRC
```

+ 配置fpm
```
user = ***
group = ***
listen = 127.0.0.1:10001
pm.max_children = 16
```

+ 安装配置nginx请参考[这篇文章](/blog/2013/03/31/nginx-pei-zhi-mo-ban/)
