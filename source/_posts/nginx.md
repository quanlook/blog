---
title: nginx 配置
date: 2020-11-25 14:00:35
tags:
  - nginx
  - 运维
categories: 
  - 运维
  - nginx
top_img: 
keywords:
description:
comments:
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359257.jpg
toc:
toc_number:
auto_open:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
---

# Nginx 配置

CentOS 安装nginx

```sh
[vagrant@master ~]$ sudo cat >> /etc/yum.repos.d/nginx.repo<<EOF
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
EOF
[vagrant@master ~]$
```

什么是 nginx ?

Nginx 是高性能的 HTTP 和反向代理的服务器，处理高并发能力是十分强大的， 能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。

## 正向代理

需要在客户端配置代理服务器进行指定网站访问。

正向代理，是在用户端的。如你要访问谷歌浏览器地址，但是你是不能直接访问的，此时你可以通过一个代理服务器来帮助你访问。

[![xphDb9.jpg](https://s1.ax1x.com/2022/09/18/xphDb9.jpg)](https://imgse.com/i/xphDb9)
比如你企业没有外网那如何访问外网查阅质料呢？

此时公司肯定有电脑是可以连接外网的，那么将我们的网络连接配置到那台服务器上，那么我们就可以访问了，相当于我们通过公司那台服务器访问查阅资料。这就是正向代理。

### 正向代理

```

```

## 反向代理

暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。
<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215546.png) -->
[![xphyU1.jpg](https://s1.ax1x.com/2022/09/18/xphyU1.jpg)](https://imgse.com/i/xphyU1)
反向代理是作用在服务器端的，是一个虚拟ip(VIP)。对于用户的一个请求，会转发到多个后端处理器中的一台来处理该具体请求。

大型网站都有DNS(域名解析服务器)，load balance(负载均衡器)等。

<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215554.png) -->
[![xphsER.jpg](https://s1.ax1x.com/2022/09/18/xphsER.jpg)](https://imgse.com/i/xphsER)

两者的对比图如下

------

<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215559.png) -->
[![xph28K.jpg](https://s1.ax1x.com/2022/09/18/xph28K.jpg)](https://imgse.com/i/xph28K)

### 反向代理配置

```sh
server {
  listen 80;
  server_name 192.168.1.4;
  location ~ /edu/ {
    proxy_pass http://localhost:8080;
  }
  
  location ~ /vod/ {
    proxy_pass http://localhost:5000;
  }

}
# http://192.168.4/edu/
# http://192.168.4/vod/
```

## 负载均衡

增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡。

<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215604.png) -->
[![xphgC6.jpg](https://s1.ax1x.com/2022/09/18/xphgC6.jpg)](https://imgse.com/i/xphgC6)
由Nginx服务器帮忙分配
<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215608.png) -->
[![xphRgO.jpg](https://s1.ax1x.com/2022/09/18/xphRgO.jpg)](https://imgse.com/i/xphRgO)

------

好处就是缓解原先一个服务器的访问压力

<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215613.png) -->
[![xphWvD.jpg](https://s1.ax1x.com/2022/09/18/xphWvD.jpg)](https://imgse.com/i/xphWvD)

### 负载均衡配置

```sh
vim /etc/nginx/nginx.conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
VPS（Virtual Private Server 虚拟专用服务器）
events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;

upstream web.com { #这个是定义Tomcat服务的负载均衡
    server 192.168.1.5:8080; #服务器A
    server 192.168.1.6:8080; #服务器B

}
server {
    listen       80;
    server_name  192.168.1.4; #宿主机ip

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / { #//这个是调用Tomcat服务的负载均衡
        proxy_pass http://web.com; #需要负载均衡的服务器
        #root /html;
        #index index.html index.php index.jsp;
    }
}

}

```

负载均衡策略

| 模式 | 备注 |
| ------------------ | --------------- |
| 轮询               | 默认方式        |
| weight             | 权重方式        |
| ip_hash            | 依据ip分配方式  |
| least_conn         | 最少连接方式    |
| fair（第三方）     | 响应时间方式    |
| url_hash（第三方） | 依据URL分配方式 |

Nginx负载均衡分配策略

- 轮询（默认）
  
- 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，会自动剔除！
  
- weight

  - weight代表权，重默认为1，权重越高被分配的客户端越多！

    指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

    比如：

```sh
upstream web.com { #这个是定义Tomcat服务的负载均衡
    server 192.168.1.5:8080 weight=5; #服务器A
    server 192.168.1.6:8080 weight=10; #服务器B
}
# 权重为10的那个会比权重为5的那个客户端多一倍！
```

- ip_hash
  - 每个请求按访问ip的hash解果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

```sh
upstream web.com { #这个是定义Tomcat服务的负载均衡
  ip_hash;
    server 192.168.1.5:8080; #服务器A
    server 192.168.1.6:8080; #服务器B
}
# 这样产生的结果：第一次这个用户被分配到了哪个服务端，以后他只会一直访问那个服务端！
```

- fair（第三方）
  - 按照后端服务器的响应时间来分配，响应时间短的优先分配！

```sh
upstream web.com { #这个是定义Tomcat服务的负载均衡
    server 192.168.1.5:8080; #服务器A
    server 192.168.1.6:8080; #服务器B
    fair;
}
```

## 动静分离

动静分离是将网站静态资源（HTML，JavaScript，CSS，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问。

动静分离是指在web服务器架构中，将静态页面与动态页面或者静态内容接口和动态内容接口分开不同系统访问的架构设计方法，进而提升整个服务访问性能和可维护性。
<!-- ![在这里插入图片描述](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025215620.png) -->
[![xphhKe.jpg](https://s1.ax1x.com/2022/09/18/xphhKe.jpg)](https://imgse.com/i/xphhKe)
nginx动静分离的好处

1、api接口服务化：动静分离之后，后端应用更为服务化，只需要通过提供api接口即可，可以为多个功能模块甚至是多个平台的功能使用，可以有效的节省后端人力，更便于功能维护。

2、前后端开发并行：前后端只需要关心接口协议即可，各自的开发相互不干扰，并行开发，并行自测，可以有效的提高开发时间，也可以有些的减少联调时间

3、减轻后端服务器压力，提高静态资源访问速度：后端不用再将模板渲染为html返回给用户端，且静态服务器可以采用更为专业的技术提高静态资源的访问速度。

动静分离配置：

```sh
server {
    listen       80;
    server_name  192.168.1.4; #宿主机ip

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location /www/ { # 动态页面
        root /data/;
        index index.html index.php index.jsp;
    }
    
    location /static/ { # 静态页面
        root /static/;
        autoindex on; # 列出当前文件夹的内容
    }
}

 # 动态页面: http://192.168.1.4/www/ 
 # 静态页面:http://192.168.1.4/static/
```

## 虚拟主机

### 虚拟主机配置

```

```

## Keepalived+Nginx+Tomcat 主备模式

<!-- ![Keepalived+Nginx+Tomcat 主备模式](https://gitee.com/quanlook/mapPictures/raw/master/images/20201025220240.png) -->
[![xphoVA.jpg](https://s1.ax1x.com/2022/09/18/xphoVA.jpg)](https://imgse.com/i/xphoVA)

**内容持续更新中….**
