---
title: ThinkPHP5 构造方法如何进行验证
date: 2019-06-03 20:17:01
tags: ['php', 'thinkphp']
category:
- php
- thinkphp
---

# 前言

平时想必都会遇到权限验证或者是路由重定向的需求，需要在构造方法做点处理

但实践中，必定会遇到同一个问题

我明明已经在构造方法做了判断，并且有return重定向response对象，它居然不重定向并且还执行了其它方法

# 解决

解决这个问题有以下方式

- 使用中间件
- 死扛构造方法

# 发现
- 实践发现，在构造方法中抛出异常，是不会其它方法
- 查看redirect源码，发现它是返回一个response对象
- 翻查其它源码后发现HttpResponseException这个好东西

# 死扛构造方法

`HttpResponseException` 类，`redirect` 方法 两者搭配使用;

```php



use think\exception\HttpResponseException;

class Index
{
    public function __construct()
    {
        if (不通过) {
          // 接收response对象
          $response = redirect('url');

          // 抛出一个http异常
          throw new HttpResponseException($response);
        }
    }
}

```