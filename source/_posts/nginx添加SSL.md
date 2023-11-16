---
title: nginx添加SSL安全证书
date: 2022-03-10 14:00:35
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
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359258.png
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

# nginx添加SSL安全证书

白嫖 `ssl`一年

- <https://freessl.cn/>

nginx配置证书

```nginx

mkdir /user/local/src/nginx/cert
[root@lhzero nginx]# cat conf/nginx.conf
user  root;
worker_processes  2;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream monitor-admin {
        server 127.0.0.1:9090;
    }
    upstream xxljob-admin {
        server 127.0.0.1:9100;
    }
    server {
        listen       443;
        server_name  lhzero.cn;
        ssl on;
        ssl_certificate cert/full_chain.pem;
        ssl_certificate_key cert/private.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        location / {
            root /root/barco-wisdom/dist;
            index  index.html index.htm;
        }
        location /admin/ {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://monitor-admin/admin/;
        }
        location /xxl-job-admin/ {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://xxljob-admin/xxl-job-admin/;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    server{
        listen 80;
        server_name lhzero.cn;
        rewrite ^/(.*)$ https://lhzero.cn:443/$1 permanent;
    }
    include vhosts/*.conf;
}
```
