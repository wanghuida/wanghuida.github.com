---
layout: post
title: "CutyCapt html页面生成图片 分享微博"
date: 2013-07-15 16:47
comments: true
sharing: true
footer: true
categories: [Share]
---

+ CutyCapt一个用Qt Webkit把html页面生成图片的工具

+ Mac下安装使用

```
brew install cuty_capt

CutyCapt --private-browsing=off --min-width=640 --min-height=440 \
    --url="http://meimeidou.cn/diySubject/detail/key/1366957139/magnify/1" --out="/tmp/img.png"
```


+ Ubuntu生产环境下安装使用

```
# xvfb是一个虚拟的X windows
sudo apt-get install xvfb
sudo apt-get install cutycapt

# 搞一个中文字体，雅黑就不错，可以自己找也可以联系我
scp msyh.ttf idc01-002:/tmp
sudo cp /tmp/msyh.ttf /usr/share/fonts/truetype/

# 如果缺少icu48可以装下
sudo apt-get install libicu48

# 生成图片
xvfb-run --server-args="-screen 0, 1024x768x24" cutycapt --private-browsing=off --min-width=640 --min-height=440 \
    --url="http://meimeidou.cn/diySubject/detail/key/1366957139/magnify/1" --out="/tmp/img.png"

```
