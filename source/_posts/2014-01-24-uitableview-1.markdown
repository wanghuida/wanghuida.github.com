---
layout: post
title: "UITableView (1)"
date: 2014-01-24 17:28
comments: true
sharing: true
footer: true
categories: [IOS]
---

### 展示一下最基础的uitableview应该如何使用，最后有完整的代码

1 自己先创建一个ViewController, 添加一个UITableView内部属性和数据

```objective-c
@interface AETableViewController ()
{
  UITableView *_table;  
  NSMutableArray *_data;
}
```

2 头里需要实现UITableViewDataSource, UITableViewDelegate（delegate其实不用，但实际一般都会用到）

```objective-c
@interface AETableViewController : UIViewController<UITableViewDataSource, UITableViewDelegate>
@end
```

3 往_data中加入一点数据,初始化_table,设置代理和数据源

```objective-c
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
  self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
  if (self) {
    _data = [NSMutableArray arrayWithArray:@[@"美美豆", @"抢拍", @"预约", @"wanghd.com", @"王惠达"]];
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  CGRect rect = [[UIScreen mainScreen] bounds];
  _table = [[UITableView alloc]initWithFrame:rect style:UITableViewStylePlain];
  _table.delegate = self;
  _table.dataSource = self;
  [self.view addSubview:_table];
}
```

<!-- more -->

4 设置数据源的总数和内容

```objective-c

#pragma mark -
#pragma mark datasource

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  return [_data count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *cellId = @"cell";
  // 这里可以复用cell提高性能
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
  if (cell == nil) {
    cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
  }
  cell.textLabel.text = [_data objectAtIndex:indexPath.row];
  return cell;
}

```

5 运行起来看下效果吧

<img style="max-width:300px;" src="/images/post/ios-uitableview-1.jpg" />

6 附赠一份完整的代码

```objective-c
// .h
#import <UIKit/UIKit.h>

@interface AETableViewController : UIViewController<UITableViewDataSource, UITableViewDelegate>

@end

// .m
#import "AETableViewController.h"

@interface AETableViewController ()
{
  UITableView *_table;
  NSMutableArray *_data;
}

@end

@implementation AETableViewController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
  self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
  if (self) {
    _data = [NSMutableArray arrayWithArray:@[@"美美豆", @"抢拍", @"预约", @"wanghd.com", @"王惠达"]];
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  CGRect rect = [[UIScreen mainScreen] bounds];
  _table = [[UITableView alloc]initWithFrame:rect style:UITableViewStylePlain];
  _table.delegate = self;
  _table.dataSource = self;
  [self.view addSubview:_table];
}


#pragma mark -
#pragma mark datasource

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  return [_data count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *cellId = @"cell";
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
  if (cell == nil) {
    cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
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
