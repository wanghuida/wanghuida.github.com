---
layout: post
title: "使用JMX分析solr状态"
date: 2012-09-20 14:25
comments: true
sharing: true
footer: true
categories: [Java, Solr]
---


###开启JMX
+ 在solrconfig.xml中添加,默认包含

```
<jmx />
```

+ 重启solr，设置启动参数
+ 扩展选项请参考[官方文档](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)

```
java -Xmx256M -Xms256M -XX:PermSize=64M -XX:MaxPermSize=64M 
-XX:+UseParallelGC 
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=3000
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
-jar start.jar
```

###使用jconsole连接，查看进程状态

```
jconsole 127.0.0.1:3000 （jconsole在jdk的bin目录里）
```

###实时java堆使用情况

![heap](/images/post/heap.png "heap")

###实时cpu使用情况

![cpu](/images/post/cpu.png "cpu")

###MBean分析
####documentCache文档缓存
