---
title: thinkphp-6-multi-app-tutorial
date: 2019-07-26 21:14:08
tags: ["php", "thinkphp"]
category:
  - php
  - thinkphp
---

# 创建多应用

以下创建两个应用，并创建对应的控制器

```
.
├── app
│   ├── project
│   │   └── controller
│   │       └── Index.php
│   ├── project2
│   │   └── controller
│   │       └── Index.php
```
## 配置
### 应用一
```php
# /project/controller/Index.php
namespace app\project\controller;

use app\BaseController;

class Index extends BaseController
{
    public function index()
    {
        return 'hello,' . 'project';
    }

    public function say()
    {
        return 'say hello,' . 'project';
    }
}
```

### 应用二

```php
# /project2/controller/Index.php
namespace app\project2\controller;

use app\BaseController;

class Index extends BaseController
{
    public function index()
    {
        return 'hello,' . 'project2';
    }

    public function say()
    {
        return 'say hello,' . 'project2';
    }
}

```

## 开启多应用

```php
# /config/app.php
'auto_multi_app' => true

```

## 运行
访问以下地址，你就可以看到已经成功了。
- 你的域名/project
- 你的域名/projec2

## 多应用 路由配置
按以下路径创建文件就行

- /route/project/xxx.php
- /route/project2/xxx.php

**重点! 重点! 重点!**

多应用的路由，路由不用再配置应用名

例如 有这么一条路由 /project/say 那配置是这样

```php
use think\facade\Route;

// 错的
// Route::rule('project/say', 'index/index/say');

// 对的
Route::rule('say', 'index/index/say');
```