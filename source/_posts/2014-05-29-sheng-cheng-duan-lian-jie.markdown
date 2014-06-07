---
layout: post
title: "生成短链接"
date: 2014-05-29 17:40
comments: true
sharing: true
footer: true
categories: [Php]
---


+ 类似于微博使用的方法，生成好短链接，在数据库里存储映射，然后获取重定向

```php
<?php

function getShortLink($long_link)
{
    # 配合md5的key
    $key = "wanghuida";
    $long_link_md5 = md5($long_link . $key);

    # 总共62的字符
    $letter_number = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    # 128位的md5 分4段计算
    $retval = array();
    for ($i = 0; $i < 4; $i++) {
        $sub_md5 = substr($long_link_md5, $i*8, 8);
        # 只要30位，头部2位砍掉
        $hex = 0x3FFFFFFF & hexdec($sub_md5);
        # 再把30位切成6份，每份都获取一个字符
        $short_link = '';
        for ($j = 0; $j < 6; $j++) {
            $index = 0x0000003d & $hex;
            $short_link .= $letter_number[$index];
            $hex = $hex >> 5;
        }
        $retval[] = $short_link;
    }

    return $retval;
}


$retval = getShortLink("http://wanghd.com/name=wanghuida&age=28");
var_dump($retval);

```


