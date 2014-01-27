---
layout: post
title: "CALayer 常用例子"
date: 2014-01-27 14:13
comments: true
sharing: true
footer: true
categories: [IOS]
---


+ 先看下默认效果

<img style="max-width:100px;" src="/images/post/cglayer1.jpg" />

```objective-c

UIImage *avatar = [UIImage imageNamed:@"touxiang"];
UIImageView *avatarView = [[UIImageView alloc]initWithImage:avatar];
avatarView.frame = CGRectMake(50, 50, 80, 80);
avatarView.center = CGPointMake(rect.size.width/2, rect.size.height/2);
[self.view addSubview:avatarView];

```

+ 第一种就圆角头像

<img style="max-width:100px;" src="/images/post/cglayer2.jpg" />

```objective-c

avatarView.layer.borderWidth = 1; //边框
avatarView.layer.borderColor = [[UIColor grayColor] CGColor]; //边框颜色
avatarView.layer.cornerRadius = 40; //圆角
avatarView.layer.masksToBounds = YES; //隐藏超出的部分

```

<!-- more -->

+ 第二种是加阴影

<img style="max-width:100px;" src="/images/post/cglayer3.jpg" />

```objective-c

avatarView.layer.shadowColor = [[UIColor blackColor] CGColor]; //颜色
avatarView.layer.shadowOffset = CGSizeMake(3, 3); //控制阴影的方向,x,y
avatarView.layer.shadowRadius = 3; //阴影范围
avatarView.layer.shadowOpacity = 0.8; //控制阴影透明度

```
