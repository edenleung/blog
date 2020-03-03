---
layout: ThinkPHP5 前后分离 session配置
title: rearseparation-session-configurtion
date: 2019-04-23 21:26:48
tags: ["php", "thinkphp", "vue"]
category:
  - php
  - thinkphp
---


### 前端工作
配置axios

```
const service = axios.create({
  // 其它代码...
  // 允许发送cookie，axios默认关闭
  withCredentials: true
})
```

### 后端工作
路由配置 或者 自动添加 header

`Access-Control-Allow-Origin` 不能为*,必须配置前端域名
`Access-Control-Allow-Credentials` 开启配置

以一条路由为例子。

```php

Route::miss('test/index')->header('Access-Control-Allow-Origin', 'http://前端域名')->header('Access-Control-Allow-Credentials', 'true')->allowCrossDomain();

```

### 生成session
`localhost:9522` 前端域名与端口，80的不用填写

```php

  // 登录成功
  session([
    'prefix'     => 'module',
    'auto_start' => true,
    'domain'     => 'localhost:9522'
  ]);

  session('user', $uid);

```

### 验证

```php

    // 验证
    session([
        'prefix'     => 'module',
        'auto_start' => true,
        'domain'     => 'localhost:9522'
    ]);

    if (session('?user')) {
        echo '已登录';
    } else {
        echo '未登录';
    }

```
