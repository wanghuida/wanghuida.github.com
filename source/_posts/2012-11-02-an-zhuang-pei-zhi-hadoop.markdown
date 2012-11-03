---
layout: post
title: "安装配置hadoop"
date: 2012-11-02 23:38
comments: true
sharing: true
footer: true
categories: [Hadoop, Java]
---


###下载stable的hadoop
+ hadoop的版本比较多，建议下载`stable`的，现在是`hadoop-1.0.4`
+ 提供一个[下载链接](http://hadoop.apache.org/releases.html#Download)


###解压缩
```
tar -zxvf hadoop-1.0.4.tar.gz
cd hadoop-1.0.4
```

###配置hadoop

> vim编辑conf/hadoop-env.sh，设置环境变量

```
#如果是mac的话，请添加下面这句
export HADOOP_OPTS="-Djava.security.krb5.realm=OX.AC.UK 
    -Djava.security.krb5.kdc=kdc0.ox.ac.uk:kdc1.ox.ac.uk"
#配置JDK，mac的关系所以路径比较飘逸
export JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home
```

<!-- more -->

> vim编译conf/core-site.xml，设置hdfs端口

```
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

> vim编辑conf/hdfs-site.xml，设置hdfs复制份数

```
<configuration>
    <property>
        <name>dfs.replication</name>  
        <value>1</value>  
    </property> 
</configuration>
```


> vim编译conf/mapred-site.xml，设置job跟踪器的端口(map-reduce的)

```
<configuration>
    <property>
        <name>mapred.job.tracker</name>
        <value>localhost:9001</value>
    </property>
</configuration>
```

###配置ssh

```
cat ~/.ssh/id_dsa.pub >> authorized_keys
```

![ssh](/images/post/ssh.jpg "ssh")

###启动测试hodoop

+ 如果端口没有被占用的应该一切OK了

```
bin/hadoop namenode -format
bin/start-all.sh
#增加一些文件，这里用你自己的目录替换
bin/hadoop fs -put ~/project/mogilefs-tool /mogilefs-tool
```

+ 在游览器里查看下结果，输入localhost:50070
