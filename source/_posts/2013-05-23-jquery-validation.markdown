---
layout: post
title: "jquery validation"
date: 2013-05-23 12:41
comments: true
sharing: true
footer: true
categories: [Share,Javascript]
---


### 表单验证在网站开发中是肯定有的，还好jquery在这方面已经做的很好了


+ 自定义一种验证类型

```javascript
jQuery.validator.addMethod(
    "cellphone",
    function(value, element) {
        return this.optional(element) || /^1\d{10}$/.test(value);
    },
    "请输入有效的手机号码"
);
```

<!-- more -->

+ 定义规则和错误显示

```javascript
$('#form').validate({
    rules: {
        nickname: "required",
        email: {
            required: true,
            email: true
        },
        password: {
            required: true,
            minlength: 6,
        }
    },
    messages: {
        nickname: "请输入昵称",
        email: {
            required: "请输入邮箱",
            email: "请输入有效的邮箱"
        },
        password: {
            required: "请输入密码",
            minlength: "密码的长度不能小于6位"
        }
    },
    errorElement: "div",
    wrapper: "div",
    errorPlacement: function(error, element) {
        element.parent().after(error);
    },
    highlight: function(element, errorClass) {
        $(element).parent().css('border', '1px solid red');
    },
    unhighlight: function(element, errorClass) {
        $(element).parent().css('border', '1px solid #777');
    },
    submitHandler: function() { alert("Submitted!") }
});
```
