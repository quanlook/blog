---
title: Docker常用应用部署
date: 2021-05-31 19:55:38
tags: docker
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359251.jpg
---





## MySQL

```sh
docker run -itdp 3307:3306  \
--name mysql \
--restart=always \
-e MYSQL_ROOT_PASSWORD=db123456 \
-e TZ=Asia/Shanghai \
-v /data/mysql:/var/lib/mysql \
mysql:5.7 \
--character-set-server=utf8 \
--collation-server=utf8_unicode_ci \
--character-set-client-handshake=FALSE
```
> MYSQL 常见变量
> MYSQL_ROOT_PASSWORD
> MYSQL_DATABASE
> MYSQL_USER, MYSQL_PASSWORD
> MYSQL_ALLOW_EMPTY_PASSWORD
> MYSQL_RANDOM_ROOT_PASSWORD
> MYSQL_ONETIME_PASSWORD
> MYSQL_INITDB_SKIP_TZINFO

## Redis

```sh
docker run -itdp 6379:6379 \
--name redis \
--privileged=true \
--restart=always \
-v /data/redis/data:/data \
redis:latest \
--requirepass "redisPassword"\ # 镜像内部命令
--appendonly yes  # 数据追加到文件

#
config set maxmemory 1024MB
config get maxmemory
```

> 注意：**--requirepass** 参数是redis参数要放在镜像的后面 

## MongDB

```sh
docker run -itdp 27017:27017 \
--name mongodb \
--restart=always \
-e MONGO_INITDB_ROOT_USERNAME=you_name \
-e MONGO_INITDB_ROOT_PASSWORD=you_password \
-v /data/mongdb/db:/data/db \
mongo:latest
```

> 官方文档：https://hub.docker.com/_/mongo?tab=description
>
>  `MONGO_INITDB_ROOT_USERNAME` 初始化用户名
>
>  `MONGO_INITDB_ROOT_PASSWORD` 初始化密码
>
> `MONGO_INITDB_DATABASE` 初始化数据库名称
>
> `MONGO_INITDB_ROOT_PASSWORD_FILE` 指定文件加载MongDB用`户名`和`密码 `

## MariaDB

```sh
docker run -itdp 3306:3306 \
--name mariadb \
-e MYSQL_ROOT_PASSWORD=db123456 \
-v /data/mariadb:/var/lib/mysql \
mariadb:latest \
--character-set-server=utf8 \
--collation-server=utf8 \
--character_set_database=utf8 \
--character_set_server=utf8 \
--character_set_system=utf8 \
--character_set_client=utf8 \
--character_set_results=utf8
```

## Nginx

```sh
docker run -idtp 80:80 \
--name nginx \
--restart=always \
-v /data/nginx:/usr/share/nginx/html \
nginx:latest
```

## OpenResty

```sh
# https://hub.docker.com/r/openresty/openresty
#  /usr/local/openresty/nginx/conf/conf.d/*.conf

docker run -itd \
-p 80:80 \
--name openresty \
--restart=always \
-v /data/openresty/html:/usr/local/openresty/nginx/html \
openresty/openresty:1.21.4.2-alpine

docker run -itd \
-p 80:80 \
--name openresty \
--restart=always \
-v /data/openresty/conf.d:/etc/nginx/conf.d \
-v /data/openresty/html:/usr/local/openresty/nginx/html \
openresty/openresty:1.21.4.2-alpine

docker run -itd \
-p 80:80 \
--name openresty \
-v /data/openresty/conf:/usr/local/openresty/nginx/conf/ \
-v /data/openresty/conf/nginx.conf:/usr/local/openresty/nginx/conf/nginx.conf：ro \
-v /data/openresty/html:/usr/local/openresty/nginx/html/
openresty/openresty:1.21.4.2-alpine
```

## Java

```dockerfile
FROM java:8
MAINTAINER quanlook
ADD demo-0.0.1-SNAPSHOT.jar demo.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","demo.jar"]
```

## Node

```dockerfile
#引用镜像
FROM node:8.9.4-alpine
#作者
MAINTAINER quanlook
#执行命令，创建文件夹
RUN mkdir -p /usr/src/workPlace_express/expressPro

#将dist目录拷贝到镜像里
COPY ./dist /usr/src/workPlace_express/expressPro/dist/
COPY package.json /usr/src/workPlace_express/expressPro
COPY server.js /usr/src/workPlace_express/expressPro

#指定工作目录
WORKDIR /usr/src/workPlace_express/expressPro

#安装依赖及构建node应用
RUN npm install
#配置环境变量
 ENV HOST 0.0.0.0
 ENV PORT 3333
#定义程序默认端口
EXPOSE 3333
#运行程序命令
CMD ["node","server.js"]
```

## Python

```dockerfile
FROM python:3.7
WORKDIR /usr/src/app
COPY requirements.txt ./
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/
COPY ./src/*  ./
CMD ["python", "server.py"] 
```

## Go

```dockerfile
.
├── Dockerfile
├── app
│   └── main
└── script
    └── build.sh
==================
FROM golang:latest
MAINTAINER  quanlook
WORKDIR /go/src/
COPY . .
EXPOSE 80
CMD ["/bin/bash", "/go/src/script/build.sh"]

```

## OwnCloud

```sh
mkdir  /data/owncloud
docker run -itdp 8081:80 \
--name owncloud \
--restart=always \
-v /data/owncloud:/var/www/html/data \
owncloud:latest
```

## NextCloud

```sh
mkdir /data/nextcloud
docker run -itdp 8080:80 \
--name nextcloud \
--restart=always  \
-v /data/nextcloud:/var/www/html/data \
nextcloud:latest
```

## RabbitMQ

```sh
docker run -itd \
--name rabbitMQ \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-p 15672:15672 \
-p 5672:5672 \
-p 25672:25672 \
-p 61613:61613 \
-p 1883:1883 \
rabbitmq:management
```

## zookeeper

```sh
docker run -itd --name zookeeper \
--publish 2181:2181 \
--volume /etc/localtime:/etc/localtime \
zookeeper:latest 
```

## Etcd

> Docker Hub: https://hub.docker.com/r/bitnami/etcd

```sh
# 参考如下，使用Docker安装ETCD
# 
docker run -itd --name Etcd-server \
    --network app-tier \
    --publish 2379:2379 \
    --publish 2380:2380 \
    --env ALLOW_NONE_AUTHENTICATION=yes \
    --env ETCD_ADVERTISE_CLIENT_URLS=http://etcd-server:2379 \
    bitnami/etcd:latest
```



## Kafka

```sh
docker run -itd \
--name kafka \
--publish 9092:9092 \
--link zookeeper \
--env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
--env KAFKA_ADVERTISED_HOST_NAME=  \
--env KAFKA_ADVERTISED_PORT=9092 \
--volume /etc/localtime:/etc/localtime \
docker.io/wurstmeister/kafka
```

## Nacos

```sh
docker run -itdp 8848:8848 \
--name nacos \
-e MODE=standalone \
--restart=always \
nacos/nacos-server
```

## ConSul

```sh
docker run -itdp 8500:8500/tcp 
--name consul \
--restart=always \
consul agent -server -ui -bootstrap-expect=1 -client=0.0.0.0
```
