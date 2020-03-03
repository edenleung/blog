---
title: API接口 签名效验
date: 2019-10-16 18:56:06
tags: [php]
category:
  - php
---


### 客户端生成
必要参数值
- token
- 时间戳
- 随机字符
- ...

```php

$params = sort([
   token值, 时间戳, 随机字符, ...更多自定义参数
], SORT_STRING);

$tmpStr = implode($params);
$sign = sha1($tmpStr);

```

### 服务端验证
```php

$signature = $data['signature'];

unset($data['signature'])

$tmpArr= array_values($data);
sort($tmpArr, SORT_STRING);
$tmpStr = implode($tmpArr);
$tmpStr = sha1($tmpStr);

if ($tmpStr == $signature) {
    return true;
} else {
    return false;
}

```