---
title: Nginx下部署ThinkPHP5应用到二级目录
date: 2019-06-10 14:09:19
tags: ["php", "thinkphp"]
category:
  - php
  - thinkphp
---

### 前言
有时可能项目上或域名资源紧缺的情况，可以一个域名下有多个项目运行。

以下就按部署ThinkPHP项目为例，将后端代码部署到二级目录

- www.domain.com 前端静态文件
- www.domain.com/api 后端源码

### 配置

要注意以下几个环节

- location
- rewrite
- alias 指定到后端项目 `public` 路径上
- fastcgi_param 不能加document_root配置

```

server {
    
    # ...其它代码

    # 匹配到以 www.domain.com/api/ 开头的url
    location /api/ {
        # 指定后端代码位置 并指向到public文件夹 具体位置请按照你实际路径填写就行
        alias /var/www/html/www.domain.com/think/public/;
        index index.php;

        # 注意下方 fastcgi_param的值
        location ~ .php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            include        fastcgi_params;
            fastcgi_param  SCRIPT_FILENAME  $request_filename;
        }

        # 必须加上rewrite 不然你的子链接全404, 必须rewrite到think文件夹，具体位置请按照你实际路径填写就行
        if (!-e $request_filename) {
            rewrite ^/think/(.*)$ /think/index.php?s=$1 last;
        }
    }  
}

```

**注意**

静态资源文件 `static` 不能带绝对路径 /
