---
title: ThinkPHP6 事件订阅例子配置
date: 2019-07-29 23:00:26
tags: ["php", "thinkphp"]
category:
  - php
  - thinkphp
---

### 前言
大概过了下ThinkPHP6文档，相比以前的tp5, 多了个订阅功能。

### 对比
#### 订阅事件(subscribe)
- 支持订阅多个事件(监听)

#### 事件监听(listener)
- 只能监听一个事件（跟TP5的行为大致一样）

### 应用

立马体验一波！

场景： 订单发货后 触发事件 完成发送短信通信

#### 注册事件/监听器

```php

// 事件定义文件
return [
    'bind'      => [
        // 绑定订单发货类
        'OrderShipped' => 'app\event\OrderShipped',
    ],

    'listen'    => [
        'AppInit'  => [],
        'HttpRun'  => [],
        'HttpEnd'  => [],
        'LogLevel' => [],
        'LogWrite' => [],

        // 监听器【订单发货】事件
        'OrderShipped' => ['app\listener\OrderShipped']
    ],

    'subscribe' => [
        // 订阅【订单】事件 此文件可订阅（监听）【订单类】事件等
        'app\subscribe\Order'
    ],
];
```

#### 绑定订单发货类

`app\event\OrderShipped`

```php


namespace app\event;

use app\Model\Order;

/**
 * 订单事件绑定类
 * 
 */
class OrderShipped
{
    /**
     * 订单对象模型(包含订单的数据)
     *
     * @var [type]
     */
    public $order;

    /**
     * 构造时 注入订单对象
     *
     * @param Order $order
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}

```

#### 监听事件
`app\listener\OrderShipped`

```php



namespace app\listener;

use app\event\OrderShipped as Event;

/**
 * 处理订单发货事件
 * 
 */
class OrderShipped
{
    public function handle(Event $event)
    {
        echo '收到事件 - {OrderShipped}' . PHP_EOL;

        // 获取订单模型 已包含订单详情的数据对象
        // echo '<pre>';
        // var_dump($event->order);

        // 现在可以使用order的信息发邮件 写日志等等
        
    }    
}

```

### 订阅订单事件
`app\subscribe\Order`

```php

namespace app\subscribe;

use app\event\OrderShipped as Event;

/**
 * 订阅订单相关事件
 * 
 */
class Order
{
    /**
     * 收到订单发货事件
     *
     * @param [type] $event
     * @return void
     */
    public function onOrderShipped(Event $event)
    {
        echo '收到订阅 - {订单发货事件}' . PHP_EOL;

        // 获取订单信息
        // echo '<pre>';
        // var_dump($event->order);
    }
}
```

### 使用

```php


use app\Model\Order;

public function orderShip()
{
    // 查找订单
    $order = Order::find(1);

    // 发货处理 更新订单状态
    // ...

    // 发布订单发货事件
    event('OrderShipped', $order);
}
```

### 重点

可能有些小伙伴有这么一个好奇，这个 `$order` 数据对象是否怎么传入到对应的处理方法，并且使用 `$event->order` 就可以获取订单信息

注意以下文件内容, 这些文件都有注入事件绑定类

`public function handle(Event $event)`

`public function onOrderShipped(Event $event)`

- `listen/OrderShipped.php`
- `subscribe/Order.php`

当调用 `event('OrderShipped', $order)` 事件时，系统分发到对应的处理方法(OrderShipped, onOrderShipped)

`OrderShipped`、 `onOrderShipped` 自动注入（反射）`OrderShipped` 事件类，并且实例化的时候也注入订单数据对象到 `OrderShipped` 成员变量 `order`

最后可以使用 `$event->order` 来获取订单信息来做短信通知等任务