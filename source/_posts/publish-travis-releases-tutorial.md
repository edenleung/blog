---
title: Travis 发布 Releases 配置
date: 2019-08-27 20:32:39
tags: ["travis"]
category:
  - travis
---

### 前言
实现 `php` 项目带composer.json配置的，想发布release时，自动下载所需要的包，供其它人下载。

**注意配置**
 - before_deploy
 - on:tags
 - overwrite

 `deploy:file` 必须在 `before_deploy` 时生成，不然travis会找不到。

### 配置

```bash
# travis.yml

before_deploy:
  - echo 'start'
  - composer self-update
  - composer install --no-dev --no-interaction --ignore-platform-reqs
  - zip -r --exclude='*.git*' --exclude='*.zip' --exclude='*.travis.yml' Project_Full.zip .

deploy:
  provider: releases
  api_key:
    secure: your_sercure_key
  file: /path/file
  skip_cleanup: true
  overwrite: true
  on:
    tags: true
```