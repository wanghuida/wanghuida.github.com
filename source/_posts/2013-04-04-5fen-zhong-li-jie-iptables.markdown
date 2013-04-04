---
layout: post
title: "5分钟理解iptables"
date: 2013-04-04 10:08
comments: true
sharing: true
footer: true
categories: [Share]
---

### iptables作用

1. 过滤请求，根据条件决定通过或拒绝
2. 网络地址转换
3. 修改头信息

### iptables的组成部分

+ table：根据类型分了3种，分别是filter,nat,mangle。filter用来过滤，nat用来地址转换，mangle用来改ttl之类的头信息【不常用】。
+ chain：是指网络包处理的某个阶段，看下图：1.yes这条路线目的地址是本机。2.no这条路线目的地址是其他网段，需要转发。

![iptables](/images/post/iptables.gif "iptables")

<!-- more -->

+ rule：用来判断哪些包需要处理，例如-p指定协议，-s指定源地址，-d指定目的地址，-i和-o定义输入输出的设备
+ target：其实用target这个词很不好理解，我的理解是符合规则后的操作


### iptables命令的语法

```
$ sudo iptables [table] [chain] [rule] [target]
# table用-t表示，例如-t nat，-t fileter。如果不加-t 默认就是filter
# chain前面需要加一个动词，例如-A INPUT意思是"添加 input链"，例如-I POSTROUTING 2意思是"插入 postrouting 到第二条记录"
# rule上面已经解释过了
# target可以是ACCEPT，DROP，SNAT，MASQUERADE等，代表满足规则的话，就执行这种操作
```

### 记录一些例子，并解释

```
# 清空nat表里的记录
sudo iptables -F -t nat
# 在filter表里增加一条都接收的记录 
sudo iptables -t filter -A INPUT -j ACCEPT
# 更改从eth1出去的源地址为114.77.99.212，一般用在共享单个ip
sudo iptables -t nat -A POSTROUTING -o eth1 -j SNAT --to-source 114.77.99.212
```
