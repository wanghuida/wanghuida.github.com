---
layout: post
title: "纯代码写ios"
date: 2014-01-22 18:30
comments: true
sharing: true
footer: true
categories: [IOS]
---


### 纯代码写ios 后期更容易维护

+ 半年没有写文章，趁过年放假好好补补，用一个简单的纯代码helloworld开始吧

+ 如果对xcode一无所知可以从官方的教程开始 <a href="https://developer.apple.com/library/ios/referencelibrary/GettingStarted/RoadMapiOSCh/chapters/Introduction.html">官方教程</a>


1 打开xcode创建一个空项目呗

<img style="max-width:500px;" src="/images/post/ios-new-project.jpg" />

2 查看AppDelegate

```objective-c
// AppDelegate 就是UIApplicationMain的代理 , 启动时会调用一下方法
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    // 初始化一个uiwindow
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    // 设置默认背景为白色
    self.window.backgroundColor = [UIColor whiteColor];
    // 显示到最前端，并获取输入
    [self.window makeKeyAndVisible];
    return YES;
}
```

<!-- more -->

3 添加一个自己view controller

```objective-c
#import "AEViewController.h"

@interface AEViewController ()

@end

@implementation AEViewController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
  self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
  if (self) {
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  CGRect rect = [[UIScreen mainScreen] bounds];
  UILabel *label = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, 100, 50)];
  label.center = CGPointMake(rect.size.width/2, rect.size.height/2);
  label.text = @"Hello World";
  label.textColor = [UIColor blackColor];
  [self.view addSubview:label];
}

- (void)didReceiveMemoryWarning
{
  [super didReceiveMemoryWarning];
}

@end

```

4 设置rootViewController

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
  self.window.backgroundColor = [UIColor whiteColor];
  [self.window makeKeyAndVisible];
  
  AEViewController *viewCtl = [[AEViewController alloc]init];
  self.window.rootViewController = viewCtl;
  
  return YES;
}
```

5 运行看一下效果

<img style="max-width:300px;" src="/images/post/ios-new-project-ret.jpg" />
