---
layout: post
title: "yii 用户验证"
date: 2013-05-24 12:39
comments: true
sharing: true
footer: true
categories: [Php]
---

1. yii用户验证其实很简单，但是网络上的文章都写的不够详细【包括官方的】
2. 先介绍一下基于session的用户验证
3. 用cookie让用户可以保持状态，再介绍一下安全性

<!-- more -->

+ main.php 配置登录页面

```php
<?php

'components' => array (
    'user' => array (
        'loginUrl' => array('user/login'),
    ),
)
```

+ 在控制器中增加权限控制和规则

```php
<?php

class UserController extends CController 
{
    # 让控制器加入到权限控制
    public function filters()
    {
        return array(
            'accessControl',
        );
    }

    # 让edit,view这2个action访客无法直接使用，跳转到登录界面
    public function accessRules()
    {
        return array(
            array('deny',
                'actions'=>array('edit', 'view'),
                'users'=>array('?'),
            ),
        );
    }

    public function actionEdit() { var_dump('edit'); }
    public function actionView() { var_dump('view'); }
}

```

+ 创建继承CUserIdentity的用户标识类给登录时使用

```php
<?php

class UserIdentity extends CUserIdentity
{
    protected $_id;

    public function authenticate()
    {
        /*
            这里可以写判断是否有用户的逻辑

            需要存储到session里的数据可以这样写
            $this->setState('nickname', $member->nickname);
        */
        $this->_id = $member->user_id;
        return true;
    }

    public function getId()
    {
        return $this->_id;
    }
}
```

+ 用户登陆的控制器

```php
<?php

class UserController extends Controller
{
    private $req;

    private $user;

    public function actionLogin()
    {
        $this->req = Yii::app()->request;
        $this->user = Yii::app()->user;

        $reason = '';
        if ($this->req->isPostRequest) {

            $email = $this->req->getPost('email','');
            $password = $this->req->getPost('password','');

            $identity = new UserIdentity($email, $password);
            if($identity->authenticate()) {
                $this->user->login($identity);
                $this->redirect($this->user->returnUrl);
            } else {
                $reason = $identity->errorMessage;
            }
        }
        $this->render('/login', array('reason'=>$reason));
    }
}

```

<hr />


+ 增加登录cookie保持用户登录状态，改完赶紧尝试一下吧

```php
<?php


# 修改main.php,允许自动登录
'user' => array (
    'allowAutoLogin' => true,
    'loginUrl' => array('user/login'),
),

# 修改UserController设置7天有效期
$this->user->login($identity, 86400);
```

+ 不经问自己：这样的cookie安全吗？会不会被利用？

```
1. 不应该把敏感信息保存到state里，比如密码
2. yii内部有cookie校验机制，改动的cookie是无法生效的，可以看下/protected/runtime/state.bin（这就是验证随机的key）
3. 想要知道yii内部生成key源码可以参考 CSecurityManager

protected function generateRandomKey()
{
    return sprintf('%08x%08x%08x%08x',mt_rand(),mt_rand(),mt_rand(),mt_rand());
}
```
