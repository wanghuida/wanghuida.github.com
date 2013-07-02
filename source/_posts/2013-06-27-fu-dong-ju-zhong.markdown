---
layout: post
title: "浮动居中"
date: 2013-06-27 10:52
comments: true
sharing: true
footer: true
categories: [Share]
---



+ 对于定宽的非浮动元素我们可以用 margin:0 auto; 进行水平居中，对于不定宽的浮动元素我们也有一个常用的技巧解决它的水平居中问题。分解如下：

+ HTML 部分：

```
<div class="test">
    <p>浮动的</p>
    <p>居中的</p>
</div>
```

+ CSS 部分：

```
.test{
    float:left;
    position:relative;
    left:50%;
}
p{
    float:left;
    position:relative;
    right:50%;
}
```

+ 这样看来就很简单了吧，父元素和子元素同时左浮动，然后父元素相对左移动50%，再然后子元素相对右移动50%，或者子元素相对左移动-50%也就可以了。如此简单如此神奇。
