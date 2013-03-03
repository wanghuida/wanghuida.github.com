---
layout: post
title: "Mysql 安全"
date: 2013-02-27 14:57
comments: true
sharing: true
footer: true
categories: [Mysql]
---


1. 如果客户端和服务器端的连接需要跨越并通过不可信任的网络，那么就需要使用SSH隧道来加密该连接的通信。一般情况下mysql不直接对外，只提供内部访问。

2. 用GRANT和REVOKE语句来控制对MySQL的访问。不要授予超过需求的权限。决不能为所有主机授权。

3. 不要让任何人(除了MySQL root账户)访问mysql数据库中的user表！这很关键。加密的密码才是MySQL中的真正的密码。

4. 不要将纯文本密码保存到数据库中。如果你的计算机有安全危险，入侵者可以获得所有的密码并使用它们。相反，应使用MD5()、SHA1()或单向哈希函数。

5. 不要从词典中选择密码。有专门的程序可以破解它们。

6. 试试从Internet使用nmap工具扫描端口。MySQL默认使用端口3306。或者telnet ip 3306。

7. 如果某个用户输入“; DROP DATABASE mysql;”等内容，应确保你的应用程序保持安全。

8. 一定要记住还应检查数字数据。如果当用户输入值234时，应用程序生成查询SELECT * FROM table WHERE ID=234，用户可以输入值234 OR 1=1使应用程序生成查询SELECT * FROM table WHERE ID=234 OR 1=1。结果是服务器查找表内的每个记录。

9. 学会使用tcpdump和strings工具。在大多数情况下，你可以使用下面的命令检查是否MySQL数据流未加密

```
shell> tcpdump -l -i eth0 -w - src or dst port 3306 | strings
```

10. 对所有MySQL用户使用密码。

```
shell> mysql -u root
mysql> UPDATE mysql.user SET Password=PASSWORD('newpwd')
    -> WHERE User='root';
mysql> FLUSH PRIVILEGES;
```

11. 绝对不要作为Unix的root用户运行MySQL服务器。这样做非常危险，因为任何具有FILE权限的用户能够用root创建文件(例如，~root/.bashrc)。为了防止，mysqld拒绝用root运行，除非使用--user=root选项明显指定。

12. 确保mysqld运行时，只使用对数据库目录具有读或写权限的Unix用户来运行。

13. 不要将PROCESS或SUPER权限授给非管理用户。mysqladmin processlist的输出显示出当前执行的查询正文，如果另外的用户发出一个UPDATE user SET password=PASSWORD('not_secure')查询，被允许执行那个命令的任何用户可能看得到。 mysqld为有SUPER权限的用户专门保留一个额外的连接，因此即使所有普通连接被占用，MySQL root用户仍可以登录并检查服务器的活动。 可以使用SUPER权限来终止客户端连接，通过更改系统变量的值更改服务的器操作，并控制复制服务器。

14. 不要向非管理用户授予FILE权限。有这权限的任何用户能在拥有mysqld守护进程权限的文件系统那里写一个文件。
