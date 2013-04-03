---
layout: post
title: "tcpdump 简明教程"
date: 2013-04-03 16:10
comments: true
sharing: true
footer: true
categories: [Share]
---


+ tcpdump作用很简单，就是抓取数据包。很常用的网络分析工具。对于理解tcp/ip协议也非常有帮助。 

```
# 请使用root用户，运行下面的命令捕获数据，你会发现数据包多的自己都不想看了
$ sudo tcpdump
```

<!-- more -->

+ tcpdump有很多选项，不过先不急着学选项。先能用起来才是关键，那先看几个`条件关键字`吧

<table class="table table-bordered" style="max-width:350px; margin-left:37px">
<tbody>
  <tr>
    <td>host</td>
    <td>可以是ip也可以是主机名</td>
  </tr>
  <tr>
    <td>net</td>
    <td>网段</td>
  </tr>
  <tr>
    <td>port</td>
    <td>端口</td>
  </tr>
  <tr>
    <td>ip,tcp,icmp,udp</td>
    <td>协议</td>
  </tr>
  <tr>
    <td>src</td>
    <td>来源</td>
  </tr>
  <tr>
    <td>dst</td>
    <td>目的地</td>
  </tr>
  <tr>
    <td>and</td>
    <td>同时满足</td>
  </tr>
  <tr>
    <td>or</td>
    <td>满足任何一个即可</td>
  </tr>
  <tr>
    <td>not</td>
    <td>非</td>
  </tr>
</tbody>
</table>

+ 学了那么多条件关键字，来几个例子用起来吧

```
# 先学习几个简单的
$ sudo tcpdump host 192.168.0.145
$ sudo tcpdump src 192.168.0.145
$ sudo tcpdump dst 192.168.0.145
$ sudo tcpdump net 192.168.0.0/24
$ sudo tcpdump tcp
$ tcpdump port 80

# 组合起来用用
$ sudo tcpdump tcp and src host 192.168.0.145 and port 80
$ sudo tcpdump ip and not dst 192.168.0.145 and dst port 80

# 加上组，可以更加复杂一些
$ sudo tcpdump tcp and host 192.168.0.145 and \(port 80 or port 8080\)

```

+ 感觉不错，加一点选项让tcpdump更加有用

<table class="table table-bordered" style="max-width:350px; margin-left:37px">
<tbody>
  <tr>
    <td>-w file-name</td>
    <td>将捕获到的数据写入文件</td>
  </tr>
  <tr>
    <td>-i device-name</td>
    <td>网络设备名，常用eth0</td>
  </tr>
  <tr>
    <td>-n</td>
    <td>不解析域名</td>
  </tr>
  <tr>
    <td>-nn</td>
    <td>不解析域名和端口</td>
  </tr>
  <tr>
    <td>-X,-XX</td>
    <td>用16进制和ascii显示包里的内容</td>
  </tr>
  <tr>
    <td>-v,-vv,-vvv</td>
    <td>越多越详细的头</td>
  </tr>
  <tr>
    <td>-c number</td>
    <td>捕获包的数量</td>
  </tr>
  <tr>
    <td>-s number</td>
    <td>获取多少内容, 0代表所有</td>
  </tr>
  <tr>
    <td>-S</td>
    <td>打印绝对的序列号</td>
  </tr>
</tbody>
</table>

+ 让选项和条件组合起来，发挥tcpdump强大威力

```
$ sudo tcpdump -vXSs 0 not dst port 8080 and \(dst net 10.0.0.0/8 or 192.168.0.0/24\)

# 一个http的数据包
17:55:51.846893 IP (tos 0x0, ttl 43, id 10976, offset 0, flags [DF], proto TCP (6), length 237)
    ec2-23-21-128-184.compute-1.amazonaws.com.http > 192.168.0.145.60897: Flags [P.], cksum 0x5e78 (correct), seq 504653705:504653890, ack 894919854, win 27, options [nop,nop,TS val 222645590 ecr 909764515], length 185
    0x0000:  4500 00ed 2ae0 4000 2b06 cb24 1715 80b8  E...*.@.+..$....
    0x0010:  c0a8 0091 0050 ede1 1e14 6789 3557 64ae  .....P....g.5Wd.
    0x0020:  8018 001b 5e78 0000 0101 080a 0d45 4d56  ....^x.......EMV
    0x0030:  3639 e7a3 4854 5450 2f31 2e31 2032 3030  69..HTTP/1.1.200
    0x0040:  204f 4b0d 0a53 6572 7665 723a 206e 6769  .OK..Server:.ngi
    0x0050:  6e78 2f30 2e37 2e36 370d 0a44 6174 653a  nx/0.7.67..Date:
    0x0060:  2057 6564 2c20 3033 2041 7072 2032 3031  .Wed,.03.Apr.201
    0x0070:  3320 3039 3a35 353a 3531 2047 4d54 0d0a  3.09:55:51.GMT..
    0x0080:  436f 6e74 656e 742d 5479 7065 3a20 696d  Content-Type:.im
    0x0090:  6167 652f 6769 660d 0a43 6f6e 7465 6e74  age/gif..Content
    0x00a0:  2d4c 656e 6774 683a 2034 330d 0a43 6f6e  -Length:.43..Con
    0x00b0:  6e65 6374 696f 6e3a 2063 6c6f 7365 0d0a  nection:.close..
    0x00c0:  0d0a 4749 4638 3961 0100 0100 8001 0000  ..GIF89a........
    0x00d0:  0000 ffff ff21 f904 0100 0001 002c 0000  .....!.......,..
    0x00e0:  0000 0100 0100 0002 024c 0100 3b         .........L..;

```

+ tcpdump还可以根据tcp里某些状态位进行过滤

```
tcpdump 'tcp[13] & 1!=0'
```

<br />
+ 附送tcp头信息图
![tcp-header](/images/post/tcp-header.jpg "tcp-header")

<br />
+ 附送ip头信息图
![ip-header](/images/post/ip-header.jpg "ip-header")
