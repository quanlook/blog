---
title: MongoDB-for-Windows-install
date: 2020-11-15 02:21:42
tags: MongoDB
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359254.jpg
categories: 
  - 数据库
---

# MongoDB install

MongoDB官网：

- <https://www.mongodb.com/try/download/database-tools>

MongoDB for Windows：

- <https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2012plus-4.2.11-rc1.zip>

```powershell
tree /f /a
D:.
|   LICENSE-Community.txt
|   MondoDBinstall.md
|   mongo.config  # 创建mongdb 配置文件
|   MPL-2
|   README
|   THIRD-PARTY-NOTICES
|   THIRD-PARTY-NOTICES.gotools
|
+---bin
|       bsondump.exe
|       Install-Compass.ps1
|       mongo.exe
|       mongo.pdb
|       mongod.exe
|       mongod.pdb
|       mongodump.exe
|       mongoexport.exe
|       mongofiles.exe
|       mongoimport.exe
|       mongorestore.exe
|       mongos.exe
|       mongos.pdb
|       mongostat.exe
|       mongotop.exe
|
+---data 
|   \---db #添加 data和db 目录
\---logs 
        mongo.log # 创建 logs目录和 mongo.log文件
```

创建数据目录

```powershell
mkdir data\db
```

创建日志目录和文件

```powershell
mkdir logs &&  touch logs/mongo.log
```

创建mongodb配置文件`mongo.config`

```powershell
#数据库路径
dbpath=D:\mongodb\data\db 
#日志输出文件路径
logpath=D:\mongodb\logs\mongo.log 
#错误日志采用追加模式
logappend=true 
#启用日志文件，默认启用
journal=true 
#这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
#quiet=true 
#端口号默认为27017
port=27017 
```

临时启动：

```powershell
mongod  --dbpath D:\mongodb\data\db  --logpath D:\mongodb\logs\mongo.log
```

> 可以在浏览器 打开：  <http://localhost:27017>  查看到数据库配置没问题可以启动
>
>
> It looks like you are trying to access MongoDB over HTTP on the native driver port.
>

安装服务：

```powershell
mongod --dbpath D:\mongodb\data\db --logpath D:\mongodb\logs\mongo.log --serviceName "MongoDB" --install
```

移除服务：

```powershell
mongod --logpath "D:\mongodb\logs\mongo.log" \
--logappend --dbpath "D:\mongodb\data\db" \
--directoryperdb --serviceName "MongoDB" \
--serviceDisplayName "MongoDB" --remove
```

重新安装:

```powershell
mongod --logpath "D:\mongodb\logs\mongo.log" \
--logappend --dbpath "D:\mongodb\data\db" \
--directoryperdb --serviceName "MongoDB" \
--serviceDisplayName "MongoDB" --install
```

Windows服务的 启动/停止：

```powershell
# 启动
net start mongodb
#停止
net stop mongodb
```
