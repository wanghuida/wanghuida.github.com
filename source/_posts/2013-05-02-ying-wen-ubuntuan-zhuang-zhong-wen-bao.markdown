---
layout: post
title: "英文Ubuntu安装中文包"
date: 2013-05-02 18:07
comments: true
sharing: true
footer: true
categories: [Share]
---


### 查看系统内安装的locale：

```
locale -a
# 如果没有“zh-CN.UTF-8”，则表示系统内没有安装中文locale。这会导致“LC_CTYPE: cannot change locale (zh_CN.UTF-8)”的警告。
```

### 输入以下命令安装：

```
cd /usr/share/locales 
sudo ./install-language-pack zh_CN
```

+ 然后重开终端，就可以发现中文locale已经安装完毕，警告已经不再出现了。
