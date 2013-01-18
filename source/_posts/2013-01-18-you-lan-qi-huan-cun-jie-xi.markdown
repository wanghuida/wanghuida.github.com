---
layout: post
title: "游览器缓存解析"
date: 2013-01-18 19:32
comments: true
sharing: true
footer: true
categories: [Php]
---


### 游览器缓存：可以通过如下四种方式进行设置

1. Expire: 直接设置过期时间(不做详细解释，用Cache-Control更好)
2. Cache-Control: HTTP 1.1用Cache-Control代替Expire，Cache-Control比Expire更方便，功能更多。不需要访问服务器端
3. Last-Modified: 用来设置最后更新时间。需要访问服务器端，如果没有更新直接使用304，节省网络流量和网络传输时间
4. ETag: 用来设置唯一标识。需要访问服务器端，和last-modified一样

<!-- more -->


### Cache-Control

```php
<?php
header("Cache-Control: max-age=120, must-revalidate");
echo "cache cache";  
```

![cache-control](/images/post/cache-control.jpg "cache-control")

+ 刷新(cmd + r)，强刷(shift + cmd + r)，都会访问服务器, 因为这两种方式游览器会绕过本地缓存。尝试打开一个新的tab访问，没有和服务器交互。
+ windows用户的刷新(F5)，强刷(ctrl + F5)

### Last-Modified

```
<?php

$time = strtotime('2013-01-18'); 
header("Last-Modified: " . gmdate("D, d M Y H:i:s", $time) . " GMT"); 
 
$modified_since = isset($_SERVER['HTTP_IF_MODIFIED_SINCE']) ? $_SERVER['HTTP_IF_MODIFIED_SINCE'] : false;
if ($modified_since) {
    $modified_since = strtotime($modified_since);
}
 
if ($time == $modified_since) {
    echo "hit"; 
} else {
    echo "no hit";
}
```

![last-modified](/images/post/last-modified.jpg "last-modified")

![last-modified-ret](/images/post/last-modified-ret.jpg "last-modified-ret")

+ 第一次访问显示no hit，刷新则命中缓存显示hit(一般304处理，我这里只是举例)。强刷则会显示no hit，可见游览器的强刷是有特别用处的

### ETag

```
<?php
 
$etag = 'test-etag';
$none_match = @$_SERVER['HTTP_IF_NONE_MATCH']; 
 
if ($none_match == $etag) {
    header("HTTP/1.1 304 Not Modified", true, 304); 
    exit;
}
 
header("ETag: {$etag}");
echo "test ok!";
```

![etag](/images/post/etag.jpg "etag")

![etag304](/images/post/etag304.jpg "etag304")

+ 第一次访问http返回200，显示test ok!。刷新http返回304，使用本地缓存，显示test ok!。强刷则绕过本地缓存，http返回200，可见和last-modified一样
