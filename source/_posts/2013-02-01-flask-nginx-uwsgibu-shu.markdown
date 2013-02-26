---
layout: post
title: "flask nginx uwsgi 部署"
date: 2013-02-01 14:10
comments: true
sharing: true
footer: true
categories: [Python]
---

### 安装 uwsgi

```
$ brew install uwsgi
```

### 安装 nginx

```
$ brew install nginx 
```

### 配置Flask环境

```
$ mkdir test
$ cd test
$ virtualevn sandbox
$ . sandbox/bin/activate
$ pip install Flask
```

<!-- more -->

### 创建一个Flask项目

```
$ touch test.py

# 内容如下

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello World'

if __name__ == '__main__':
    app.run()

```

### 创建uwsgi配置文件

```
$ touch test_config.xml

<uwsgi>
    <pythonpath>/Users/william/project/test</pythonpath>
    <module>test</module>
    <callable>app</callable>
    <socket>/tmp/uwsgi.sock</socket>
    <master/>
    <processes>4</processes>
    <memory-report/>
</uwsgi>

```

### nginx 配置

```
server {
    listen 80;
    server_name www.test.com;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/tmp/uwsgi.sock; 
    }
}
```

### 启动服务

```
sudo nginx -s reload

# 支持virtualenv, uswgi还可以--py-auto-reload
sudo uwsgi -x /Users/william/project/test/test_config.xml --virtualenv sandbox 

```
