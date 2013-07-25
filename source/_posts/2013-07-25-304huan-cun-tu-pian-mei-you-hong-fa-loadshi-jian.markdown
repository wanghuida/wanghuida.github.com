---
layout: post
title: "304缓存图片没有触发load事件"
date: 2013-07-25 14:54
comments: true
sharing: true
footer: true
categories: [Javascript]
---


+ 图片墙需要计算图片高度，只有等图片load好计算的高度才正确，所以代码如下：

```javascript

$(".img-div").each(function(){
    
    //设置load事件
    $(this).find('img').load(function(){
        /* 用height()获取高度，省略事件逻辑 */
    });

});

```

+ 感觉上面的代码很符合逻辑了，问题出现了：

+ 当图片返回304缓存时，该图片不能触发load事件。。。（通过判断complete属性来修正）

```javascript

    $(".img-div").each(function(){

        $(this).find('img').one('load', function(){
            /* 用height()获取高度，省略事件逻辑 */

            //使用one强制事件只触发一次，缓存的图片通过load()方法进行触发

        }).each(function() {
          if(this.complete) $(this).load();
        });
    });
```
