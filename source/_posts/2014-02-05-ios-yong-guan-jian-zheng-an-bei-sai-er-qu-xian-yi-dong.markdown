---
layout: post
title: "ios 用关键帧按贝塞尔曲线移动"
date: 2014-02-05 19:36
comments: true
sharing: true
footer: true
categories: [IOS]
---

### 先看下效果，黑色方块延黑色线条移动

<img style="max-width:300px;" src="/images/post/keyframe.jpg" />

<!-- more -->


1 使用CGMutablePathRef勾划移动路径，并显示出来

```objective-c
CGFloat x = 50;
CGFloat y = 50;
CGMutablePathRef path = CGPathCreateMutable();
CGPathMoveToPoint(path, nil, x, y);
CGPathAddLineToPoint(path, nil, 300, 300);
CGPathAddLineToPoint(path, nil, 50, 300);
CGPathAddLineToPoint(path, nil, 50, 50);
CGPathAddLineToPoint(path, nil, 300, 300);

// 二次贝塞尔曲线
CGPathAddQuadCurveToPoint(path, nil, 50, 50, 50, 300);
// 三次贝塞尔曲线
CGPathAddCurveToPoint(path, nil, 200, 200, 150, 400, 300, 300);
```

+ 不了解贝塞尔曲线函数的看下图可以加深体会

<img style="max-width:300px;" src="/images/post/Bezier2.gif" />
<br />
<img style="max-width:300px;" src="/images/post/Bezier3.gif" />
<br />



2 先按path把路径画出来，其实是张图

```objective-c
UIGraphicsBeginImageContext(self.view.frame.size);
CGContextRef context = UIGraphicsGetCurrentContext();
CGContextAddPath(context, path);
[[UIColor blackColor] setStroke];
CGContextDrawPath(context, kCGPathStroke);
UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();

UIImageView *imgView = [[UIImageView alloc]initWithFrame:self.view.bounds];
imgView.image = img;
[self.view addSubview:imgView];
```

3 画个小方块

```objective-c
UIView *view = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 30, 30)];
view.backgroundColor = [UIColor blackColor];
[self.view addSubview:view];
```

4 给小方块增加移动效果

```objective-c

// CAKeyframeAnimation 就是关键帧的动画
CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
// 设置移动路径
animation.path = path;
animation.duration = 5.0;
// 动画结束后不还原
animation.removedOnCompletion = NO;
animation.fillMode = kCAFillModeForwards;

[view.layer addAnimation:animation forKey:@"path"];
CGPathRelease(path);
```
