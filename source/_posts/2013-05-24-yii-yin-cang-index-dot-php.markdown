---
layout: post
title: "yii 隐藏index.php RESTful nginx配置"
date: 2013-05-24 14:36
comments: true
sharing: true
footer: true
categories: [Php]
---

1. 默认yii不使用RESTful的url，比如: index.php/news/list?category=12
2. 通过简单的配置可以美化url，比如：/news/list/category/12

+ 修改main.php配置

```php
<?php

'components' => array (
    'urlManager'=>array(
        'urlFormat'=>'path',
        'showScriptName'=>false,
        'rules'=>array(),
    ),
)

```

+ 修改nginx.php配置

<!-- more -->

```
server {
    listen          11111;
    index           index-test.php index.html;
    root            /xxxxxx/xxxxx/project/yii-test;
    charset         utf-8;

    client_max_body_size 20m;

    # 是否存在文件，不存在就rewrite index-test.php，生产应该是index.php
    location / {
        if (!-e $request_filename){
            rewrite ^/(.*) /index-test.php last;
        }
    }

    # 缓存资源文件
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|swf)$ {
        expires 24h;
    }

    # 定义path_info，并交给fpm处理
    location ~ \.php {
        fastcgi_split_path_info     ^(.+\.php)(/.*)$;
        fastcgi_param               PATH_INFO                               $fastcgi_script_name;
        include                     /usr/local/etc/nginx/fastcgi.conf;
        fastcgi_pass                127.0.0.1:10001;
        fastcgi_index               index-test.php;
        expires                     off;
    }

}
```
