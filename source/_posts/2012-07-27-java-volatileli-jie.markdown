---
layout: post
title: "java volatile理解"
date: 2012-07-27 21:55
comments: true
sharing: true
footer: true
categories: [Java]
---



###volatile的含义其实很好理解，希望配合图可以更加清晰的解释

![volatile](/images/post/volatile.jpg "volatile 图例说明")

+ 多个线程加载同一个对象时，会从主内存复制一个副本到线程工作内存
+ 如果对象没有volatile属性，只有write操作才会让主内存的对象得到修改（其他线程这时才有可能拿到最新的结果）
+ 如果对象有volatile属性，load,use,asign,store都会拿到实时的结果（因为会一直同步更新结果）
