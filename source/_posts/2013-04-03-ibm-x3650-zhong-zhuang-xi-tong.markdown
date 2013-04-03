---
layout: post
title: "ibm x3650 重装系统"
date: 2013-04-03 16:02
comments: true
sharing: true
footer: true
categories: [Share]
---


### 从机房把5年前的x3650拖回来重装系统全过程

+ 原本是freebsd，做了raid1。现在希望改成ubuntu，不做raid，因为提供mogilefs服务


1. 开机启动根据提示Ctrl + A进入raid管理
2. 删除raid退出
3. 开始安装ubuntu。。。发现找不到硬盘
4. 难道没有raid就不行？再进到raid管理
5. 进行硬盘初始化，然后创建raid使用简单卷Volume即可
6. 一路安装终于没有问题了
