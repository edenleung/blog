---
title: ThinkPHP 6 RPC 例子教程
date: 2019-11-13 22:12:56
tags: [php, thinkphp]
category:
 - php
 - thinkphp
---

**RPC 依赖Swoole环境**

### 安装
```bash
composer require topthink/think-swoole
```

### 创建接口
```php
namespace app\service;

interface OperationInterface
{
    /**
     * 加法运算
     *
     * @param [type] $a
     * @param [type] $b
     * @return void
     */
    public function add($a, $b);
}
```

### 创建服务
```php
namespace app\service;

/**
 * 这是个运算服务
 * 
 */
class OperationService implements OperationInterface
{
    /**
     * 执行加法运算
     *
     * @param [type] $a
     * @param [type] $b
     * @return void
     */
    public function add($a, $b)
    {
        return $a + $b;
    }
}
```

### 配置

`swoole.php`

```php
'rpc'        => [
    'server' => [
        'enable'   => true,
        'port'     => 9000,
        'services' => [
            'OperationService' => \app\service\OperationService::class 
        ],
    ]
],
```

### 调用
```php

namespace app\controller;

use app\BaseController;
use think\swoole\rpc\client\Client;

/**
 * @package app\controllers
 * 
 */
class Index extends BaseController
{
    
    public function index(OperationService $service)
    {
        $service = $service->add([1, 2]);
        var_dump($service);
    }
}

```

### 启动
```php
php think swoole
```

访问 http://localhost:9000
