---
layout: post
title: "fpm的所有进程都占用 出现假死"
date: 2014-06-10 22:17
comments: true
sharing: true
footer: true
categories: [Share]
---

+ 查看fpm日志会看下类似于下面这种警告

```
WARNING: [pool www] seems busy (you may need to increase pm.start_servers, or pm.min/max_spare_servers), spawning 8 children, there are 0 idle, and      16 total children

```

+ 表面上看是提示你增加max children，但其实是代码写烂了


```php
<?php

    private function _curl()
    {
        $latitude = Input::get('latitude');
        $longitude = Input::get('longitude');
        $domain = Config::get('common.baidu_map_url');
        $timeout = Config::get('common.curl_timeout');
        $url = "{$domain}&location={$latitude},{$longitude}&key=meimeidou";
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, $timeout);
        curl_setopt($ch, CURLOPT_TIMEOUT, $timeout); //如果不加这句，超时后就会导致fpm无法释放
        $data = curl_exec($ch);
        curl_close($ch);
        return $data;
    }

```
