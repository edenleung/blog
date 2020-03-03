---
title: 利用 Git Diff，实现补丁（增量）包
date: 2019-09-01 20:25:13
tags: [git]
category:
  - git
---

### 前言
传统的FTP文件上传，程序更新代码时，如果文件夹深度超过两层，并且要更新的文件很多的时候，是不是觉得要疯掉了？
当然，你也可以一次性覆盖整个项目根目录。

除此之外，系统在线更新服务也需要这种补丁包的方式来更新。
当然，你也可以一次性下载源程序，进行覆盖。

以下得有些git的基础。

### 版本差异

以下按release包简单说下具体要点
生成补丁包，需要使用git diff命令。
有两个 `release` 版本

- v1.0.0
- v1.0.1

命令详情 `git diff` 旧版本 新版本

对比两个版本的代码差异，可以看到 `v1.0.0` 跟 `v1.0.1` 差异之后的文件路径，把差异之后的路径压缩成一个包，就可以实现补丁包。

```bash
git diff v1.0.0 v1.0.1 --name-only
```

### 如何找

上面已经提到如何将两个版本进行对比，但如何找到最新的版本跟之前上一个版本呢？


### 上一版本
```bash
git tag | awk '{a[NR]=$0} END{print a[NR-1]}'
```

### 最新版本
```bash
git tag | awk "END{print}"
```


## 完整例子
xargs zip update.zip 把差异结果的文件路径，进行zip压缩生成补丁包。

```bash
git diff $(git tag | awk '{a[NR]=$0} END{print a[NR-1]}') $(git tag | awk "END{print}") --name-only | xargs zip update.zip
```