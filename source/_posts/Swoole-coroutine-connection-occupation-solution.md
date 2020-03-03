---
title: Swoole 协程连接占用解决方法
date: 2019-08-03 17:38:41
tags: ["php", "swoole"]
category:
  - php
  - swoole
---

# 前言

在使用协程中，新手一般都会遇到以下报错，多个协程使用【已占用】的连接导致

> Fatal error: Uncaught SwooleError: Socket#4 has already been bound to another coroutine#505, reading of the same socket in coroutine#506

## Channel
使用Channel（管道）解决以上的问题。

```php

<?php

// 开启20个通道(可以理解为20个连接)
$chan = new co\Channel(20);

// 主协程
go(function() use(&$chan){

    // mysql服务
    $swoole_mysql = new Swoole\Coroutine\MySQL();
    $swoole_mysql->connect([
        'host' => 'mysql56',
        'port' => 3306,
        'user' => 'root',
        'password' => '1234',
        'database' => 'test',
    ]);

    // 子协程 - 保存数据到数据库
    go(function() use(&$chan, $swoole_mysql){
        while(true) {
            // 从Channel中拿出一个数据
            // Channel必须要拿数据，不然20个管道全占用了
            $data = $chan->pop();

            // 执行保存
            $swoole_mysql->query();
        }
    });

    // 创建多个子协程 - 模拟创建保存任务
    for($i = 0; $i < 20; $i++) {
        go(function() use(&$chan, $i){
            // 检查Channel(管道)是否已满
            while ($chan->isFull()) {
                // Channel(管道)已满
                // 进行休眠0.1秒等待空闲
                co::sleep(0.1);
            }

            // Channel空闲了，可以进行业务
            // ...其它业务代码

            // 可以传入一些自定义参数到Channel中
            $chan->push(['job' => $i]);
        });
    }
});

```

## 了解

主要使用Channel来控制协程什么时间执行业务代码

- $chan->push
- $chan->pop
- $chan->isFull()