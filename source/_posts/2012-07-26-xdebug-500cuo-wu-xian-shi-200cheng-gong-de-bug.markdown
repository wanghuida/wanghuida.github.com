---
layout: post
title: "xdebug:500错误显示200成功的bug"
date: 2012-07-26 16:15
comments: true
sharing: true
footer: true
categories: [Share]
---


###今天同事遇见一个诡异的问题

```php
<?php

ini_set('display_errors',0);

#随便调用一个什么方法来产生错误
abcd();
```
+ 明显php错误，但是HTTP返回200


###最终关闭加载xdebug一切正常了,返回500错误了
