---
layout: post
title: "mogstored使用nginx"
date: 2012-10-13 11:37
comments: true
sharing: true
footer: true
categories: [Mogilefs]
---

###配置nginx.conf
+ 使用mogile用户，如果使用root会导致delete job无法删除(因为新文件是root的呀)

```
user mogile daemon
```

+ autoindex on: 必须要加，否则mogadm check显示错误，因为返回错误状态码
+ expires max: 过期头设置到2037年, and the Cache-Control max-age to 10 years. 
+ client_max_body_size 20m: 放宽上传大小
+ client_body_temp_path xxxx: 存放零时文件的目录

+ 配置WebDav,首先要确认HttpDavModule已经被加载，默认是不加载的 ./configure --with-http_dav_module
+ dav_methods PUT DELETE MKCOL; 允许的方法put是上传，delete是删除,mkcol是创建文件夹
+ dav_access user:rw group:r all:r; 设置权限
+ create_full_put_path on; put新文件默认只能在已存在的目录，不过这条指令允许创建所有的中间目录

```
server 
{
    listen 7500;
    error_log /var/log/nginx/mogstore.error.log crit;
    access_log /var/log/nginx/mogstore.access.log gzip;

    location / {
        autoindex   on;
        root        /huida/mogdata;
    }

    location /dev1/ {
        expires             max;
        root                /huida/mogdata;
        client_max_body_size        20m;
        client_body_temp_path       /huida/mogdata/dev1/temp;
        dav_methods                 PUT DELETE MKCOL;
        create_full_put_path        on;
        dav_access                  user:rw group:r all:r;
    }
}
```

###配置mogstored.conf
+ 已经不需要perlbal了
+ 虽然是不用server了，不过mogstored还是要启动，因为还有个磁盘监控7501端口

```
#httplisten = 0.0.0.0:7500
server = none
```

###结论
+ nginx比perlbal更稳定，基本不会挂掉了 
+ mogadm device add 时不要忘记配置nginx后重启，也别忘记自己建文件夹【nginx多一步】
+ nginx配置好user,所以不会创建的文件权限为root:root,不用nginx就要用su mogile -c mogstored了
