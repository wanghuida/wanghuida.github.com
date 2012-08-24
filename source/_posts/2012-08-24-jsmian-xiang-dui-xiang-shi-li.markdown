---
layout: post
title: "js面向对象实例"
date: 2012-08-24 17:57
comments: true
sharing: true
footer: true
categories: [Javascript]
---

```javascript
(function(window){

var staff = function(name,phone){
    return new staff.fn.init(name,phone);
};
staff.fn = staff.prototype = {
    //静态变量
    company: '安居客',
    //构造函数
    init: function(name,phone){
        //私有变量
        var cellphone = phone;
        //公共变量
        this.username = name;
        //公共方法
        this.getPhone = function() { 
            return cellphone; 
        }
        this.setPhone = function(phone) {
            cellphone = phone;
            _test_private_function();
            return this;
        }
        //私有方法
        var _test_private_function = function() {
            console.log('private function:' + cellphone);
        }
    },
    //公共方法
    setName: function(name){
        this.username = name;
        return this;
    },
    setCompany: function(c_name){
        staff.fn.company = c_name;
        return this;
    }
};
staff.fn.init.prototype = staff.fn;
//公共方法
staff.fn.getName = function() {
    return this.username;
}
staff.fn.getCompany = function() {
    return staff.fn.company;
}
window.s = staff;
})(window);

var staff1 = s('王惠达1','11111111');
var staff2 = s('王惠达2','22222222');
//test static argument
console.log('s1:' + staff1.setCompany('瑞庭').getCompany());
console.log('s2:' + staff2.getCompany());
//test public argument
console.log('s1:' + staff1.setName('王惠达').getName());
console.log('s2:' + staff2.getName());
//test private argument
console.log('s1:' + staff1.setPhone('13712345678').getPhone());
console.log('s2:' + staff2.getPhone());
//报错：console.log('s2:' + staff2.cellphone);
```
