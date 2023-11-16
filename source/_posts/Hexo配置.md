---
title: Hexo配置
date: 2020-01-02 21:44:37
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359255.jpg
tags:
 - Blog
 - Hexo
---

# Hexo 配置

## 一、安装Nodejs和Git

**windows平台**

安装Git版本控制器

官网： <https://git-scm.com/downloads>

淘宝源镜像站：<https://npm.taobao.org/mirrors/git-for-windows>

```shell
https://npm.taobao.org/mirrors/git-for-windows/v2.29.2.windows.1/Git-2.29.2-64-bit.exe
```

安装Nodejs

官网：

- <https://nodejs.org/en/download/>

- <https://nodejs.org/dist/>

国内网：

- <http://nodejs.cn/download/>

```
https://npm.taobao.org/mirrors/node/v14.14.0/node-v14.14.0-x64.msi
```

**Linux平台**

安装Git

```shell
# Centos
sudo yum -y install git
# Ubuntu 
sudo apt -y install git

[root@localhost ~]# git --version
git version 1.8.3.1
```

安装Nodejs

```shell
[root@localhost ~]#  wget -c https://npm.taobao.org/mirrors/node/v14.14.0/node-v14.14.0-linux-x64.tar.xz

[root@localhost ~]#  -xzvf node-v14.14.0-linux-x64.tar.xz -C /usr/local/
# 1.直接添加软连接
ln -s /usr/local/node-v14.14.0-linux-x64/bin/node  /usr/local/bin/node
ln -s /usr/local/node-v14.14.0-linux-x64/bin/npm  /usr/local/bin/npm

# 2.添加系统环境变量
[root@localhost ~]# vim /etc/profile
export NODE_HOME=/usr/local/node-v14.14.0-linux-x64
export NODE_PATH=$NODE_HOME/lib/node_modules
export PATH=$PATH:$NODE_HOME/bin 

source /etc/profile # 生效环境变量

# 安装完成
[root@cloudserver ~]# node -v
v6.17.1
[root@cloudserver ~]# npm -v
3.10.10
[root@cloudserver ~]# 

```

Git 版本控制器 相关教程

- Gitee： <https://gitee.com/all-about-git>
- bilibili：
- <https://www.bootcss.com/p/git-guide/>

安装Hexo

```shell
npm install hexo-cli
```

初始化博客

```shell
# 初始化blog这个文件夹为博客目录
hexo init blog
```

运行博客

```
hexo server
# 可简写为
hexo s
```

新建一篇文章

```shell
hexo new 我的第一篇文章
# 可简写为
hexo n 文章名称
```

## 二、美化博客

选择主题

- **[hexo-theme-butterfly](https://github.com/jerryc127/hexo-theme-butterfly)**
- **[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)**
- **[hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery)**

标签外挂

```markdown
{% tabs test3, -1 %} 
<!-- tab -->
 **This is Tab 1.** 
<!-- endtab -->

<!-- tab -->
 **This is Tab 2.** 
<!-- endtab -->

<!-- tab -->
 **This is Tab 3.** 
<!-- endtab -->
 {% endtabs %}
===
{% tabs 标签, -1 %} 
<!-- tab -->
 **This is Tab 1.** 
<!-- endtab -->

<!-- tab -->
 **This is Tab 2.** 
<!-- endtab -->

<!-- tab -->
 **This is Tab 3.** 
<!-- endtab -->
 {% endtabs %}
```

```
{% tabs test4 %} 
<!-- tab第一个Tab -->
 **tab名字为第一个Tab** 
<!-- endtab -->

<!-- tab @fab fa-apple-pay -->
 **只有图标没有Tab名字** 
<!-- endtab -->

<!-- tab炸弹@fas fa-bomb -->
 **名字+icon** 
<!-- endtab -->
 {% endtabs %}


作者: Jerry
連結: https://butterfly.js.org/posts/4aa8abbe/#Tabs
來源: Butterfly
著作權歸作者所有。商業轉載請聯絡作者獲得授權，非商業轉載請註明出處。
```

```
{% tabs Book %} 
```

<!-- tab 活着 -->

## 活着

```
[![B5GgL6.jpg](https://s1.ax1x.com/2020/11/07/B5GgL6.jpg)](https://imgchr.com/i/B5GgL6)
```

 作者: [余华](https://book.douban.com/search/余华)
出版社: 作家出版社
出版年: 2012-8-1
页数: 191
定价: 20.00元
装帧: 平装
丛书: [余华作品（2012版）](https://book.douban.com/series/1634)
ISBN: 9787506365437

内容简介 ：

​ 《活着(新版)》讲述了农村人福贵悲惨的人生遭遇。福贵本是个阔少爷，可他嗜赌如命，终于赌光了家业，一贫如洗。他的父亲被他活活气死，母亲则在穷困中患了重病，福贵前去求药，却在途中被国民党抓去当壮丁。经过几番波折回到家里，才知道母亲早已去世，妻子家珍含辛茹苦地养大两个儿女。此后更加悲惨的命运一次又一次降临到福贵身上，他的妻子、儿女和孙子相继死去，最后只剩福贵和一头老牛相依为命，但老人依旧活着，仿佛比往日更加洒脱与坚强。

​ 《活着(新版)》荣获意大利格林扎纳•卡佛文学奖最高奖项（1998年）、台湾《中国时报》10本好书奖（1994年）、香港“博益”15本好书奖（1994年）、第三届世界华文“冰心文学奖”（2002年），入选香港《亚洲周刊》评选的“20世纪中文小说百年百强”、中国百位批评家和文学编辑评选的“20世纪90年代最有影响的10部作品”。

```
<!-- endtab -->

<!-- tab test@fab fa-apple-pay -->
 **只有图标没有Tab名字** 
<!-- endtab -->

<!-- tab 炸弹 -->
 **名字+icon** 
<!-- endtab -->
 {% endtabs %}
```

# Docker 发布hexo 静态网站

hexo 生成静态文件

```shell
hexo generate
# 简写
hexo g
```

发布

```shell
docker run -itdp 80:80 --name hexo-blog --restart=always  \
-v $(pwd)/public:/usr/share/nginx/html:ro \  #hexo 生成的静态页面目录
 nginx:latest
```

> -t tty 交互终端
>
> -d 后台运行
>
> -p 指定端口
>
> --name 容器名
>
> --restart=always 容器down了自动重启
>
> -v 文件映射到容器内部

优化Nginx配置

```shell
[root@cloudserver ~]# cat code/hexo-statics-blog/nginx_config/nginx.conf 

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


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

    gzip               on;
    gzip_vary          on;
    gzip_comp_level    6;
    gzip_buffers       16 8k;
    gzip_min_length    1000;
    gzip_proxied       any;
    gzip_disable       "msie6";
    gzip_http_version  1.0;
    gzip_types         text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript;

    include /etc/nginx/conf.d/*.conf;

}
[root@cloudserver ~]#
```

启动

```shell
docker run -itdp 80:80 --name hexo-blog --restart=always  \
-v $(pwd)/public:/usr/share/nginx/html:ro \
-v $(pwd)/nginx_config:/etc/nginx:ro  nginx:latest
```

```shell
docker run -itdp 80:4000 --name hexo-blog --restart=always  \
-v $(pwd)/blog:/app:rw \
docker.io/spurin/hexo:latest
```

**内容持续更新中….**
