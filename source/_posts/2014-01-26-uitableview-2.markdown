---
layout: post
title: "UITableView (2)"
date: 2014-01-26 09:30
comments: true
sharing: true
footer: true
categories: [IOS]
---

### 展示一下最基础的group table应该如何使用（一般设置中最常用），最后有完整的代码

1 自己先创建一个ViewController, 添加一个UITableView内部属性和数据

```objective-c
@interface AETableViewController ()
{
  UITableView *_table;  
  NSMutableArray *_data;
}
```

2 头里需要实现UITableViewDataSource, UITableViewDelegate

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
    NSArray *group1 = @[@"关注", @"美美豆", @"安居客"];
    NSArray *group2 = @[@"wanghd.com", @"惠达", @"达达"];
    NSArray *group3 = @[@"喜欢", @"关注", @"粉丝"];
    _data = [[NSMutableArray alloc]initWithArray:@[group1, group2]];
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  CGRect rect = [[UIScreen mainScreen]bounds];
  _table = [[UITableView alloc]initWithFrame:rect style:UITableViewStyleGrouped];
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

// 每个section head的高度 
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
  return 20;
}

// 每个section foot高度，设置为0没有效果，0.01才行
- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section
{
  return 0.01;
}

// cell的高度
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
  return 44.0;
}

// 每个section head 的view
- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section
{
  CGRect rect = CGRectMake(0, 0, 320, 20);
  UILabel *txt = [[UILabel alloc]initWithFrame:rect];
  txt.text = [NSString stringWithFormat:@"group%d", section];
  txt.textAlignment = NSTextAlignmentCenter;
  txt.font = [UIFont systemFontOfSize:11];
  return txt;
}

// section 总共有多少个
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
  return [_data count];
}

// 每个section 有多少行
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  NSArray *group = [_data objectAtIndex:section];
  return [group count];
}

// 具体的cell
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *cellId = @"cell";
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
  if (cell == nil) {
    cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
  }
  NSArray *group = [_data objectAtIndex:indexPath.section];
  cell.textLabel.text = [group objectAtIndex:indexPath.row];
  return cell;
}
```

5 测试最常用的行点击

```objective-c

#pragma mark -
#pragma mark delegate

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
  NSArray *group = [_data objectAtIndex:indexPath.section];
  NSString *txt = [group objectAtIndex:indexPath.row];
  NSLog(@"%@", txt);
}

```

6 看下效果吧

<img style="max-width:300px;" src="/images/post/ios-uitableview-2.jpg" />

7 看下完整的代码

```objective-c

// .h
#import <UIKit/UIKit.h>

@interface AEGroupTableViewController : UIViewController<UITableViewDataSource, UITableViewDelegate>

@end

// .m
#import "AEGroupTableViewController.h"

@interface AEGroupTableViewController ()
{
  UITableView *_table;
  NSMutableArray *_data;
}

@end

@implementation AEGroupTableViewController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
  self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
  if (self) {
    NSArray *group1 = @[@"关注", @"美美豆", @"安居客"];
    NSArray *group2 = @[@"wanghd.com", @"惠达", @"达达"];
    NSArray *group3 = @[@"喜欢", @"关注", @"粉丝"];
    _data = [[NSMutableArray alloc]initWithArray:@[group1, group2, group3]];
  }
  return self;
}

- (void)viewDidLoad
{
  [super viewDidLoad];
  CGRect rect = [[UIScreen mainScreen]bounds];
  rect.origin.y = 20;
  _table = [[UITableView alloc]initWithFrame:rect style:UITableViewStyleGrouped];
  _table.delegate = self;
  _table.dataSource = self;
  [self.view addSubview:_table];
}

#pragma mark -
#pragma mark datasource

- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
  return 20;
}

- (CGFloat)tableView:(UITableView *)tableView heightForFooterInSection:(NSInteger)section
{
  return 0.01;
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
  return 44.0;
}


- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section
{
  CGRect rect = CGRectMake(0, 0, 320, 20);
  UILabel *txt = [[UILabel alloc]initWithFrame:rect];
  txt.text = [NSString stringWithFormat:@"group%d", section];
  txt.textAlignment = NSTextAlignmentCenter;
  txt.font = [UIFont systemFontOfSize:11];
  return txt;
}


- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
  return [_data count];
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  NSArray *group = [_data objectAtIndex:section];
  return [group count];
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  static NSString *cellId = @"cell";
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellId];
  if (cell == nil) {
    cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellId];
  }
  NSArray *group = [_data objectAtIndex:indexPath.section];
  cell.textLabel.text = [group objectAtIndex:indexPath.row];
  return cell;
}

#pragma mark -
#pragma mark delegate

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
  NSArray *group = [_data objectAtIndex:indexPath.section];
  NSString *txt = [group objectAtIndex:indexPath.row];
  NSLog(@"%@", txt);
}



- (void)didReceiveMemoryWarning
{
  [super didReceiveMemoryWarning];
}

@end

```


