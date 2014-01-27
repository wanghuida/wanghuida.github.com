---
layout: post
title: "ios 静态库"
date: 2014-01-27 20:28
comments: true
sharing: true
footer: true
categories: [IOS]
---

### 不是真想要写静态库，只是对自己使用第三方静态库很有好处

+ 如果经常遇到undefine symbols for architecture xxx 这样的错误，那么看完后就能知道原因了

1 建静态库工程

<img style="max-width:300px;" src="/images/post/ioslib1.jpg" />

2 加一个库方法

```objective-c
// .h
+ (void)printHelloWorld;

// .m
+ (void)printHelloWorld
{
  NSLog(@"Hello World!!!");
}

```

3 直接debug运行，右键进入finder复制.h 和.a文件到新项目中

<img style="max-width:300px;" src="/images/post/ioslib2.jpg" />

<!-- more -->


4 在新项目中使用静态库 报错Undefined symbols for architecture i386:

```objective-c

#import "tmpLib.h"

[tmpLib printHelloWorld];

```

5 查看.a文件支持的架构

```
lipo -detailed_info libtmpLib.a

// 输出 Non-fat file: libtmpLib.a is architecture: armv7s
```

6 要虚拟机跑起来必须包含i386架构，还应该包含arm64 armv7 armv7s

```
// i386对应虚拟机 arm是各种机型
// debug运行生成i386架构 
// release运行生成所有arm架构

// 合并2个库
lipo -create Debug-iphonesimulator/libtmpLib.a Release-iphoneos/libtmpLib.a -output libtmpLib.a
lipo -info libtmpLib.a

// 输出Architectures in the fat file: libtmpLib.a are: i386 armv7 armv7s arm64
```

7 替换.a文件看下结果

<img style="max-width:200px;" src="/images/post/ioslib3.jpg" />
