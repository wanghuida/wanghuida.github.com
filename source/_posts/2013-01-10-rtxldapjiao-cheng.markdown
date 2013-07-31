---
layout: post
title: "RTXLDAP教程"
date: 2013-01-10 10:14
comments: true
sharing: true
footer: true
categories: [Share]
---

### 名词解释
1. RTX: 腾讯出的企业沟通工具
2. LDAP: 关于人或者其他信息的集中存储，微软的AD就是LDAP的一种
3. RTXLDAP: 基于RTX的SDK开发出的第三方工具，可以同步LDAP用户，可以RTX与LDAP单点登陆。支持RTX最新版本

<!-- more -->

### 配置使用

+ 获取源码

```
git clone https://github.com/wanghuida/RTXLDAP.git
```

+ 建议使用vs 2008打开

![rtxldap1](/images/post/rtxldap1.jpg "rtxldap1")


+ 修改App.config配置文件

```
<!-- 域 -->
<add key="DomainName" value="192.168.1.100/DC=home,DC=net" />
<!-- 域用户,用来读取所有域用户信息 -->
<add key="DomainUser" value="wanghuida" />
<!-- 域用户密码 -->
<add key="DomainPwd" value="123456" />

<!-- RTX服务IP，建议在同一台,可以免去其他配置 -->
<add key="RTXIP" value="127.0.0.1"/>
<!-- RTX服务端口 -->
<add key="RTXPort"  value="8006"/>

<!-- 标识，随意修改，不重复即可 -->
<add key="AppGUID" value="{193947E5-E990-4af8-A954-D358B385F099}"/>
<!-- 标识，随意修改，不重复即可 -->
<add key="AppName" value="WanghuidaRtxLdap"/>
```

+ F5启动调试

![rtxldap2](/images/post/rtxldap2.jpg "rtxldap2")
