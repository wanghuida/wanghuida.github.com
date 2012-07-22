---
layout: post
title: "python 一周体验记"
date: 2012-07-22 10:27
comments: true
sharing: true
footer: true
categories: [Python]
---

###python可以快速上手
>推荐官方的新手教程 [http://docs.python.org/tutorial/](http://docs.python.org/tutorial/)

<!-- more -->

###python强制空白
>这是python特有的，代码可维护性大大提高了

###python语法糖
>很甜很甜

###列表推导,列表范围
>虽然列表的处理还不如perl那么灵活，但已经有那么点意思了

>但是学习成本上python要优于perl

###PyPI
>类似于CPAN是一样的

###python在web上的应用
>python的框架有很多，可以在这个地址上找到绝大部分 [http://www.wsgi.org/en/latest/frameworks.html](http://www.wsgi.org/en/latest/frameworks.html)

>我选择了flask，同事的建议是比较小巧，好用

>可以深入了解一下Werkzeug,flask是基于它开发的

>>之前一直有个疑惑，每个框架都有内置的http server，这个http server到底是python提供的还是框架提供的? 看了一些Werkzeug的源码才清楚。Werkzeug里的serving.py使用了python的BaseHTTPServer库，Werkzeug对http进行分析和封装。So 我觉得Werkzeug是可以深入研究的。

