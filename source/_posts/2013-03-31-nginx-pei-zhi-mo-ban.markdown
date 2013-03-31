---
layout: post
title: "nginx 配置模板"
date: 2013-03-31 16:02
comments: true
sharing: true
footer: true
categories: [Share]
---


### nginx服务器端配置，可以适当调整:

+ 调整fastcgi 和 lb 的配置
+ 上传文件大小

<!-- more -->

```
user  william wheel;
# 与内核数量相等即可
worker_processes  4;

error_log  /usr/local/log/nginx/error.log  notice;
pid        /usr/local/log/nginx/nginx.pid;


events {
    worker_connections  10240;
    use epoll;
}


http {

    server_names_hash_bucket_size 64;

    include       mime.types;
    default_type  application/octet-stream;

    log_format  main '$request_time $upstream_response_time $remote_addr $request_length $upstream_addr  [$time_local] '
                      '$host "$request" $status $bytes_sent '
                      '"$http_referer" "$http_user_agent" "$gzip_ratio" "$http_x_forwarded_for" - "$server_addr"';

    access_log  /usr/local/log/nginx/access.log  main;

    # 单单作为lb可以关闭sendfile
    sendfile        on;
    tcp_nopush      on;

    # 单单作为lb应该调低，例如5或10，避免空闲连接过多
    keepalive_timeout  20;

    # 单单作为lb不要开启gzip
    gzip  on;

    # 限制内容大小，如果上传文件比较大，可以调整
    client_max_body_size  2m;
    client_body_buffer_size 1m;

    charset utf-8;

    # 反向代理
    proxy_buffering  on;
    proxy_buffers 400 256k;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # fastcgi
    fastcgi_connect_timeout 30;
    fastcgi_send_timeout 30;
    fastcgi_read_timeout 30;
    fastcgi_buffers 200 256k;

    include /usr/local/etc/nginx/conf.d/*.conf;
}

```
