---
layout: post
title: "ubuntu keychain 使用"
date: 2013-07-18 22:33
comments: true
sharing: true
footer: true
categories: [Share]
---


+ ssh-keygen生成密钥，经常会设置一个密码，让试图窃取的人无计可施。
+ 问题来了，每次登录都会要求输入密码，是时候来搞个keychain了



```bash

$ sudo apt-get install keychain
$ /usr/bin/keychain ~/.ssh/id_rsa

# 加到.profile
. ~/.keychain/$HOSTNAME-sh
. ~/.profile


```

+ keychain到底干了什么呀？

keychain会起一个ssh-agent，后来登录的人通过$HOSTNAME-sh设置环境变量使用同一个ssh-agent




