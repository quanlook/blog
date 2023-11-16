---
title: Redis-install
date: 2022-12-17 22:21:42
tags: 
  - Redis
keywords: "redis"
cover: 
toc: True
categories: 
  - 数据库
  - NoSQL
  - 缓存
---


# Redis安装

> Redis官网: <https://redis.io/download>
>
> Redis for Windows: <https://github.com/MicrosoftArchive/redis/releases>
>
> Redis Github地址: <http://download.redis.io/releases/redis-5.0.4.tar.gz>

## Redis install for Windoes

```vb
# 下载
curl -o Redis-x64-3.0.504.zip https://github.com/microsoftarchive/redis/releases/download/win-3.0.504/Redis-x64-3.0.504.zip
# 安装服务
redis-server --service-install redis.windows-service.conf  --loglevel verbose
# 卸载服务
redis-server --service-uninstall
# 启动服务
net start redis
```

## Redis 编译安装

下载，提取和编译Redis：

```shell
wget http://download.redis.io/releases/redis-5.0.4.tar.gz
tar xzf redis-5.0.4.tar.gz
cd redis-5.0.4
make
```

现在编译的二进制文件在`src` 目录中可用 。运行Redis：

```shell
src/redis-server
```

您可以使用内置客户端与Redis进行交互：

```shell
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```
