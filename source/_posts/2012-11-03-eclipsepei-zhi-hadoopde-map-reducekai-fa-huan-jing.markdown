---
layout: post
title: "eclipse配置hadoop的map-reduce开发环境"
date: 2012-11-03 00:14
comments: true
sharing: true
footer: true
categories: [Hadoop, Java]
---


###生成eclipse环境

+ 安装hadoop请参考[这篇文章](/blog/2012/11/02/an-zhuang-pei-zhi-hadoop/)

```
cd hadoop-1.0.4

#肯定会遇到诸多问题
ant eclipse 
```

> 问题1：configure: error: C++ preprocessor "/lib/cpp" fails sanity check

>> mac没有安装Xcode的command tool

> 问题2：can't exec "glibtoolize"

>> brew install autoconf automake libtool

> 问题3: .eclipse.templates does not exist

>> 自己在hadoop-1.0.4下创建一个mkdir .eclipse.templates


<!-- more -->

###编译hadoop-eclipse-plugin

<br />
1.进入eclipse插件目录

```
cd src/contrib/eclipse-plugin
```

2.编辑build.properties 

```
vim build.properties
# 设置属性，eclipse.home根据自己的定义
eclipse.home = /Applications/eclipse
version = 1.0.4
```

3.编辑build.xml

```
# 在target=jar里面增加下面的jar包
<copy file="${hadoop.root}/lib/commons-lang-2.4.jar"  todir="${build.dir}/lib" verbose="true"/>
<copy file="${hadoop.root}/lib/commons-configuration-1.6.jar"  todir="${build.dir}/lib" verbose="true"/>
<copy file="${hadoop.root}/lib/jackson-core-asl-1.8.8.jar"  todir="${build.dir}/lib" verbose="true"/>
<copy file="${hadoop.root}/lib/jackson-mapper-asl-1.8.8.jar"  todir="${build.dir}/lib" verbose="true"/>
<copy file="${hadoop.root}/lib/commons-httpclient-3.0.1.jar"  todir="${build.dir}/lib" verbose="true"/>
```

4.编辑vim META-INF/MANIFEST.MF

```
# 修改Bundle-ClassPath
Bundle-ClassPath: classes/,
  lib/hadoop-core.jar,
  lib/commons-configuration-1.6.jar,
  lib/commons-httpclient-3.0.1.jar,
  lib/commons-lang-2.4.jar,
  lib/jackson-core-asl-1.8.8.jar,
  lib/jackson-mapper-asl-1.8.8.jar

```

5.复制hadoop核心包, 并生成我们需要的插件

```
cp {root}/hadoop-core-1.0.4.jar {root}/build
#生成jar包
ant jar
```

6.编译成功后在build/contrib/eclipse-plugin下找到hadoop-eclipse-plugin-1.0.4.jar

###eclipse配置

+ 复制hadoop-eclipse-plugin-1.0.4.jar到eclipse的plugins下，然后重启eclipse

+ Preferences => Hadoop Map/Reduce

![hadoop-eclipse-location.png](/images/post/hadoop-eclipse-location.png "hadoop-eclipse-location.png")

+ Window => Show View => Other... => Map/Reduce Locations => New Hadoop Location...

![hadoop-eclipse-general.png](/images/post/hadoop-eclipse-general.png "hadoop-eclipse-general.png")

+ 看下结果吧

![hadoop-eclipse-result.png](/images/post/hadoop-eclipse-result.png "hadoop-eclipse-result.png")
