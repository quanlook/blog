---
title: Docker部署Nacos集群
date: 2020-11-29 01:11:39
tags:
  - Nacos
  - Docker
  - HA
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359250.jpg
---



Nacos单机模式：

```sh
docker run --name nacos -itdp 8848:8848 \
--privileged=true --restart=always \
-e JVM_XMS=512m -e JVM_XMX=2048m \
-e MODE=standalone \ # 指定单机模式
-e PREFER_HOST_MODE=hostname \
docker.io/nacos/nacos-server:latest

```

mysql

```sh
docker run -idp 3306:3306 \
--name mysq_nacos \
-e MYSQL_ROOT_PASSWORD=nacos \
-e TZ=Asia/Shanghai  mysql:5.7 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci \
--default-time_zone='+8:00'
```

nacos集群

- nacos-cluster.yml

```yml
version: "3"
services:
  nginx:
    container_name: nginx_proxy
    image: docker.io/nginx:latest
    volumes:
      - ../nginx_config:/etc/nginx
      - ../nginx_log:/var/log/nginx
    ports:
      - "80:80"
    restart: always
    depends_on:
      - nacos1
      - nacos2
      - nacos3
  nacos1:
    hostname: nacos1
    container_name: nacos1
    image: nacos/nacos-server:latest
    volumes:
      - ./cluster-logs/nacos1:/home/nacos/logs
      - ./init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8848:8848"
      - "9555:9555"
    env_file:
      - ../env/nacos-hostname.env
    restart: always
    depends_on:
      - mysql
  nacos2:
    hostname: nacos2
    image: nacos/nacos-server:latest
    container_name: nacos2
    volumes:
      - ./cluster-logs/nacos2:/home/nacos/logs
      - ./init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8849:8848"
    env_file:
      - ../env/nacos-hostname.env
    restart: always
    depends_on:
      - mysql
  nacos3:
    hostname: nacos3
    image: nacos/nacos-server:latest
    container_name: nacos3
    volumes:
      - ./cluster-logs/nacos3:/home/nacos/logs
      - ./init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8850:8848"
    env_file:
      - ../env/nacos-hostname.env
    restart: always
    depends_on:
      - mysql
  mysql:
    container_name: mysql
    image: nacos/nacos-mysql:5.7
    env_file:
      - ../env/mysql.env
    volumes:
      - ./mysql:/var/lib/mysql
    ports:
      - "3306:3306"
  prometheus:
    container_name: prometheus
    image: prom/prometheus:latest
    volumes:
      - ./prometheus/prometheus-standalone.yaml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    depends_on:
      - nginx
      - nacos1
      - nacos2
      - nacos3
    restart: on-failure
  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    ports:
      - 3000:3000
    restart: on-failure
```

nginx负载均衡

```sh
docker run -idtp 80:80 \
 --name nginx \
 --restart=always \
 -v nginx_config:/etc/nginx:ro \
 nginx:latest
```

添加nginx负载均衡配置`nginx_config/conf.d/default.conf`

```conf
upstream nacos_web.com {
    server 192.168.100.131:8848;
    server 192.168.100.131:8849;
    server 192.168.100.131:8850;
}
server {
    listen       80;
    server_name  192.168.100.131;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_buffering off;
        proxy_pass http://nacos_web.com;
        client_max_body_size 1024m;
    }
}
```
