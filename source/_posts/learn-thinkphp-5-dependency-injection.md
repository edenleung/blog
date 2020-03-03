---
title: 研究ThinkPHP5源码，学习到如何依赖注入
date: 2019-06-03 21:13:53
tags: ["php", "thinkphp"]
category:
  - php
  - thinkphp
---

### 前言

在实际开发ThinkPHP5的时候，【控制器】、【模型】两者的参数传递，通过 use class就能在方法里面使用到其实例化的类，这上我十分好奇它是怎么实现的

```php
use think\Request;

class User
{
    public function hello(Request $request)
    {
        $request->balabala();
    }
}
```

### 发现
[ReflectionClass](https://www.php.net/manual/zh/class.reflectionclass.php)

查看了源码发现，它是使用一个叫【ReflectionClass】反射类来实现

1. 通过反射类获取一个类的信息
2. 获取类的某个方法
3. 获取某个方法的参数信息，参数如果是一个类的话，系统自动帮你实例化
4. 最后实例化控制器，执行目标方法 并且把参数带上


### 自己实现一个

```php

<?php

class User
{
    public function say()
    {
        return 'hello';
    }
}

class Test
{
    public function sayHello(User $user)
    {
        return $user->say();
    }
}

// 使用反射类获取 `Test`类
$class = new ReflectionClass('Test'); 

// 获取Test类的`syHello`方法
$method = $class->getMethod('sayHello');

// 获取`SayHello`方法中的参数
$parameters = $method->getParameters();

$args = [];
foreach($parameters as $item) {
    
    // 获取类
    $ReflectionClass = $item->getClass();
    
    // 获致类名
    $ReflectionClass = $ReflectionClass->getName();
    
    // 实例化
    $args[] = new $ReflectionClass;
}

// 实例化`Test`类
$instance  = $class->newInstanceArgs(); 

// 执行 Test类中的`sayHello`方法 并且把上面参数带上
var_dump($method->invokeArgs($instance, $args));


```