---
layout: post
title: "date函数的误用"
date: 2013-01-06 17:22
comments: true
sharing: true
footer: true
categories: [Php]
---

### 需求：得到给定日期为一年中的第几周，并取得该周的年份

+ 产生bug的写法

```php
<?php
$time_2012_12_31 = mktime(0,0,0,12,31,2012); 
$week_2012_12_31 = date('W',$time_2012_12_31);
echo "2012-12-31: week is {$week_2012_12_31} \n";
 
$error_year_2012_12_31 = date('Y',$time_2012_12_31);
echo "2012-12-31: error year is {$error_year_2012_12_31} \n";

#输出错误的结果
2012-12-31: week is 01 
2012-12-31: error year is 2012 
```

+ 正确的写法，统一采用ISO-8601标准

```php
<?php
$time_2012_12_31 = mktime(0,0,0,12,31,2012); 
$week_2012_12_31 = date('W',$time_2012_12_31);
echo "2012-12-31: week is {$week_2012_12_31} \n";
 
$year_2012_12_31 = date('o',$time_2012_12_31);
echo "2012-12-31: year is {$year_2012_12_31} \n";

#输出正确的结果
2012-12-31: week is 01 
2012-12-31: year is 2013 
```
