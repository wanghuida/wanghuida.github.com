---
layout: post
title: "uitableview (3)"
date: 2014-01-27 13:32
comments: true
sharing: true
footer: true
categories: [IOS]
---



### tableview还有一种编辑状态也很常用，很简单下面是教程

1 老规矩 自己先创建一个ViewController, 添加一个UITableView内部属性和数据

```objective-c
@interface AEEditTableViewController ()
{
  UITableView *_table;
  NSMutableArray *_data;
}
```


2 头里需要实现UITableViewDataSource, UITableViewDelegate

```objective-c
#import <UIKit/UIKit.h>

@interface AEEditTableViewController : UIViewController<UITableViewDataSource, UITableViewDelegate>

@end
```

3 让window的root view设置为创建的view，并设置一个navigation

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
  self.window.backgroundColor = [UIColor whiteColor];
  [self.window makeKeyAndVisible];
  
  AEEditTableViewController *viewCtl = [[AEEditTableViewController alloc]initWithNibName:nil bundle:nil];
  UINavigationController *nav = [[UINavigationController alloc]initWithRootViewController:viewCtl];
  
  self.window.rootViewController = nav;
  
  return YES;
}
```
<!-- more -->

4 往_data中加入一点数据,初始化_table,设置代理和数据源，在导航上加个按钮

```objective-c
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
  self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
  if (self) {
    _data = [[NSMutableArray alloc]initWithArray:@[@"1",@"2",@"3",@"4",@"5",@"6",@"7",@"8"]];
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  
  UIBarButtonItem *btn = [[UIBarButtonItem alloc] initWithTitle:@"编辑"
                                                          style:UIBarButtonItemStyleBordered
                                                         target:self
                                                         action:@selector(tapEditBtn:)];
  self.navigationItem.rightBarButtonItem = btn;
  
  CGRect rect = [[UIScreen mainScreen] bounds];
  _table = [[UITableView alloc]initWithFrame:rect style:UITableViewStylePlain];
  _table.delegate = self;
  _table.dataSource = self;
  [self.view addSubview:_table];
}

#pragma mark - tapEditBtn

- (void)tapEditBtn:(UIBarButtonItem *)btn
{
  if ([btn.title isEqualToString:@"编辑"]) {
    btn.title = @"保存";
    [_table setEditing:YES animated:YES];
  } else {
    btn.title = @"编辑";
    [_table setEditing:NO animated:YES];
  }
}
```

5 设置数据源，增加删除的操作

```objective-c
#pragma mark - datasource

// 可以不写噢，默认就是delete模式
- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath
{
  // 当然也可以insert 就是不太用
  // return UITableViewCellEditingStyleInsert;
  return UITableViewCellEditingStyleDelete;
}

// 删除的操作
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{
  if (editingStyle == UITableViewCellEditingStyleDelete) {
    [_data removeObjectAtIndex:indexPath.row];
    [tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
  }
}

// 哪些可以删，哪些不能删
- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath
{
  return indexPath.row == 1 ? NO : YES;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  return [_data count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *cellId = @"cell";
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
  if (cell == nil) {
    cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
  }
  cell.textLabel.text = [_data objectAtIndex:indexPath.row];
  return cell;
}
```

6 看下效果吧

<img style="max-width:300px;" src="/images/post/ios-uitableview-3.jpg" />

7 附赠全部源代码

```objective-c

// .h
#import <UIKit/UIKit.h>

@interface AEEditTableViewController : UIViewController<UITableViewDataSource, UITableViewDelegate>

@end

// .m
#import "AEEditTableViewController.h"

@interface AEEditTableViewController ()
{
  UITableView *_table;
  NSMutableArray *_data;
}

@end

@implementation AEEditTableViewController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
  self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
  if (self) {
    _data = [[NSMutableArray alloc]initWithArray:@[@"1",@"2",@"3",@"4",@"5",@"6",@"7",@"8"]];
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  
  UIBarButtonItem *btn = [[UIBarButtonItem alloc] initWithTitle:@"编辑"
                                                          style:UIBarButtonItemStyleBordered
                                                         target:self
                                                         action:@selector(tapEditBtn:)];
  self.navigationItem.rightBarButtonItem = btn;
  
  CGRect rect = [[UIScreen mainScreen] bounds];
  _table = [[UITableView alloc]initWithFrame:rect style:UITableViewStylePlain];
  _table.delegate = self;
  _table.dataSource = self;
  [self.view addSubview:_table];
}

#pragma mark - tapEditBtn

- (void)tapEditBtn:(UIBarButtonItem *)btn
{
  if ([btn.title isEqualToString:@"编辑"]) {
    btn.title = @"保存";
    [_table setEditing:YES animated:YES];
  } else {
    btn.title = @"编辑";
    [_table setEditing:NO animated:YES];
  }
}


#pragma mark - datasource

- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath
{
  // 当然也可以insert 就是不太用
  // return UITableViewCellEditingStyleInsert;
  return UITableViewCellEditingStyleDelete;
}

- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{
  if (editingStyle == UITableViewCellEditingStyleDelete) {
    [_data removeObjectAtIndex:indexPath.row];
    [tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
  }
}

- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath
{
  return indexPath.row == 1 ? NO : YES;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  return [_data count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *cellId = @"cell";
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
  if (cell == nil) {
    cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
  }
  cell.textLabel.text = [_data objectAtIndex:indexPath.row];
  return cell;
}

- (void)didReceiveMemoryWarning
{
  [super didReceiveMemoryWarning];
}

@end

```
