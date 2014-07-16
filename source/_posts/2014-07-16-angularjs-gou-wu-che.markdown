---
layout: post
title: "AngularJS 购物车"
date: 2014-07-16 10:45
comments: true
sharing: true
footer: true
categories: [Javascript]
---

### 老规矩先下购物车的例子，想一下用jquery实现是需要多少绑定事件呀，然后再对比看看angularjs

<br />

<iframe width="100%" height="300" src="http://jsfiddle.net/wanghuida/vAAvw/1/embedded/result,js,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


```html
<div ng-app="CartApp">
    <div ng-controller="CartController">
         <h1>购物车</h1>

        <table>
            <tr>
                <th>商品</th>
                <th>数量</th>
                <th>单价</th>
                <th>总价</th>
                <th>删除</th>
            </tr>
            <tr ng-repeat="item in items">
                <td>{{ item.title }}</td>
                <td>
                    <input ng-model="item.quantity" />
                </td>
                <td>{{ item.price | currency }}</td>
                <td>{{ item.price * item.quantity | currency }}</td>
                <td>
                    <button ng-click="remove(index)">删除</button>
                </td>
            </tr>
            <tr>
                <th></th>
                <th></th>
                <th></th>
                <th>{{ totalPirce() | currency }}</th>
                <th></th>
            </tr>
        </table>
    </div>
</div>
```

+ ng-app定义了一个angularjs module的作用范围
+ ng-controller定义了一个控制器
+ ng-repeat 循环items
+ {{ item.price | currency }} 是一种货币的过滤器，自己写一个过滤器也是可以的
+ ng-click 是点击执行的方法


```javascript

angular.module('CartApp', [])
    .controller('CartController', function ($scope) {
    $scope.items = [
        { title: '薯片', quantity: 2, price: 5.40 }, 
        { title: '巧克力', quantity: 1, price: 12.40 }, 
        { title: '牛奶', quantity: 1, price: 45.82 }
    ];

    $scope.remove = function (index) {
        $scope.items.splice(index, 1);
    };

    $scope.totalPirce = function () {
        var total = 0;
        angular.forEach($scope.items, function (value, key) {
            total += value.quantity * value.price;
        });
        return total;
    };
});
```

+ angular.module定义模块范围
+ .controller开始一个controller的定义，这里有一个依赖注入$scope，是当前controller的作用范围，用来存储模型传递数据 $scope.items就是一个model
+ 接下来$scope里定义了2个方法remove和totalPrice
