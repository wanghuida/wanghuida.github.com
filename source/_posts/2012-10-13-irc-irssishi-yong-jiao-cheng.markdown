---
layout: post
title: "IRC:irssi使用教程"
date: 2012-10-13 23:39
comments: true
sharing: true
footer: true
categories: [Share]
---

###IRC是什么？
+ IRC（Internet Relay Chat的缩写，“因特网中继聊天”）是一种通过网络的即时聊天方式。其主要用于群体聊天，但同样也可以用于个人对个人的聊天。

<!-- more -->

###安装irssi
+ 命令行的聊天客户端满酷，就选择了irssi

```bash
brew install irssi
```

###连接服务器
+ 运行irssi
+ 连接irc.freenode.net

```bash
irssi
/connect irc.freenode.net
```

###注册一个自己用户名

```
/nick wanghuida
/msg nickserv help #会打开一个频道，Ctrl+N/P切换看一下命令,参数错误会有提示
/msg nickserv register 123456 wanghuida258@126.com
/msg NickServ VERIFY REGISTER wanghuida xgticwehprrf #这条命令是从邮箱里获取的
/msg nickserv identify 123456 #用于登陆识别 
```

###进入聊天室
+ 可以开始欢快的聊天了

```
/join #mogilefs
/join #gentoo-cn
```

###设置自动登陆
+ 也可以不通过命令行来自动登陆
+ irssi的默认配置在~/.irssi/config里

```
alias irssi_williamwang='irssi --connect=irc.freenode.net --nick=williamwang --password=xxxxxx'
```

###其他命令

+ /wc 或者 /leave ,离开当前频道
+ /disconnect <服务器>，断开一个服务器
+ /quit 或者 /exit，退出 irssi，结束IRC会话。
+ /msg <昵称> <消息>，向某人发私消息（新开窗口）
+ /query <昵称> <消息>，向某人发私消息（新开窗口且转换到这个窗口）
+ /say <昵称> <消息>，向某人说话（不新开窗口）
+ /notice <昵称> <消息>，向指定人发出注意消息
+ /me <动作>，在当前聊天室窗口中做出动作。 如做出晕倒动作：/me 晕倒
+ /away <原因>，留下信息说明暂时离开，别人向你发出私聊时将会返回此消息，再重新输入 /away（不指定参数）则解除离开状态。
+ /ignore <昵称>，忽略某人的聊天内容
+ /set autolog on，自动保存聊天记录
+ /msg ChanServ info #mogilefs ，查看聊天室信息，用来确认聊天室是否存在
+ /msg chanserv help ，创建频道之类的操作可以参考下帮助
