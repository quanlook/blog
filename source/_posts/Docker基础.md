---
title: Docker
date: 2020-11-25 12:18:11
tags: docker
top_img: 
categories: docker
keywords:
description:
comments:
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359252.png
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
---


## 一、Docker install

> <https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/>

如果你之前安装过 docker，请先删掉

```sh
sudo yum remove docker docker-common docker-selinux docker-engine
```

安装一些依赖

```sh
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

根据你的发行版下载repo文件: CentOS/RHEL Fedora

```sh
wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
```

把软件仓库地址替换为 TUNA:

```sh
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
```

安装docker-ce:

```sh
sudo yum makecache fast
sudo yum install docker-ce
# 设置开机启动
systemctl enable docker && systemctl restart docker
```

开启linux内核转发

```sh
# 开启内核转发
cat >>/etc/sysctl.conf<<EOF
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
EOF
# 加载配置
sysctl -p
```

docker拉取镜像加速

```sh
#阿里云加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": 
  [
  "https://dockerhub.azk8s.cn",
  "https://reg-mirror.qiniu.com",
  "https://registry.docker-cn.com",
  "http://hub-mirror.c.163.com",
  "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

免sudo

```sh
# 如果还没有 docker group 就添加一个：
sudo groupadd docker
# 将用户加入该 group 内。然后退出并重新登录就生效啦。
sudo gpasswd -a ${USER} docker
# 重启 docker 服务
sudo service docker restart
# 切换当前会话到新 group 或者重启 X 会话
newgrp - docker
```

努力更新中......

---

## 二、Docker常见镜像部署

部署registry镜像仓库

```sh
# 指定registry私有仓库服务器的IP地址及端口
# 修改 insecure-registries 的值，提供私有仓库的主机的ip地址和端口
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "insecure-registries":
  [
    "youHostIp/domainName:5000"
  ],
  "registry-mirrors": 
  [
  "https://dockerhub.azk8s.cn",
  "https://reg-mirror.qiniu.com",
  "https://registry.docker-cn.com",
  "http://hub-mirror.c.163.com",
  "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF
 
docker run -d -p 5000:5000 --restart=always --name registry docker.io/registry:latest

# 重新加载配置 重新启动docker服务
systemctl daemon-reload && systemctl restart docker  
# 查看私有仓库中的镜像
curl 192.168.45.129:5000/v2/_catalog
# 查看私有仓库中的镜像
curl 192.168.45.129:5000/v2/centos/tags/list

```

部署harbor镜像仓库

```sh
# harbor的Github地址
https://github.com/goharbor/harbor/releases
# 下载
wget -Oc https://github.com/goharbor/harbor/releases/download/v2.1.4/harbor-offline-installer-v2.1.4.tgz /opt/harbor-offline-installer-v2.1.4.tgz

```

部署mysql

```sh
docker run -itdp 3307:3306  --name mysql \
--restart=always \
-e MYSQL_ROOT_PASSWORD=db123456 \
-e TZ=Asia/Shanghai \
-v /data/mysql:/var/lib/mysql \
mysql:5.7 \
--character-set-server=utf8 \
--collation-server=utf8_unicode_ci \
--character-set-client-handshake=FALSE

# mysql常见变量
MYSQL_ROOT_PASSWORD
MYSQL_DATABASE
MYSQL_USER, MYSQL_PASSWORD
MYSQL_ALLOW_EMPTY_PASSWORD
MYSQL_RANDOM_ROOT_PASSWORD
MYSQL_ONETIME_PASSWORD
MYSQL_INITDB_SKIP_TZINFO
```

部署redis

```sh
docker run -itdp 6379:6379 \
--name redis \
--privileged=true \
--restart=always \
-v /data/redis/data:/data \
redis:latest \
--requirepass "redisPassword"
# 注意：--requirepass 参数是redis参数要放在镜像的后面
```

部署nginx

优雅的用docker命令

```sh
# 根据标签显示镜像
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
# 格式化输出镜像 批量配置标签
docker images --format "table docker tag {{.ID}} {{.Repository}}:{{.Tag}}" 
# 批量push到镜像仓库
docker images --format "table docker push {{.Repository}}:{{.Tag}}" 
# 提高容器的权限
docker run -tid --name centos \
--privileged \
docker.io/centos:7  /usr/sbin/init

# Docker容器的重启策略：
> 参考 https://docs.docker.com/engine/reference/run/
no 默认策略，在容器退出时不重启容器
on-failure 在容器非正常退出时（退出状态非0），才会重启容器
on-failure:3 在容器非正常退出时重启容器，最多重启3次
always 在容器退出时总是重启容器
unless-stopped 在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

```

镜像构建

Dockerfile

容器编排

docker-compose

kubenetes (k8s)

LAMP
这里我先介绍，第一种。

```shell
docker search -s 10 lamp  #搜索被收藏或使用较多的LAMP镜像，小伙伴们都推荐使用tutum/lamp
docker pull docker.io/tutum/lamp  #下载镜像
docker images
```

创建LAMP容器

```shell
mkdir /mysql_data
docker run -d --name=lamp -p 8080:80 -p 3306:3306  -v /mysql_data:/var/lib/mysql docker.io/tutum/lamp 
```

将宿主机的目录“/mysql_data”映射到容器的“/var/lib/mysql”目录。这是因为默认情况下数据库的数据库文件和日志文件都会存放于容器的AUFS文件层，这不仅不使得容器变得越来越臃肿，不便于迁移、备份等管理，而且数据库的性能也会受到影响。因此建议挂载到宿主机的目录到容器内。
接下来进入容器

```shell
# docker exec -it lamp /bin/bash
# mysql_secure_installation  //初始化数据库
```

按下回城键你会看见结尾如下的对话。
Enter current password for root (enter for none):<–初次运行直接回车
Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
New password: <– 设置root用户的密码
Re-enter new password: <– 再输入一次你设置的密码
Remove anonymous users? [Y/n] <– 是否删除匿名用户，回车
Disallow root login remotely? [Y/n] <–是否禁止root远程登录,回车
Remove test database and access to it? [Y/n] <– 是否删除test数据库，回车
Reload privilege tables now? [Y/n] <– 是否重新加载权限表，回车

All done! If you’ve completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!
初始化MariaDB完成，接下来测试登录

好的现在你只要将你的模板导入即可，至于详细设置不同网站有所不同
安装typecho个人博客模板

```shell
apt update
cd /var/www/html
rm -rf *
apt install -y wget
wget http://typecho.org/downloads/1.1-17.10.30-release.tar.gz
tar zxf  -C type 1.1-17.10.30-release.tar.gz
mv type/*  ./
cd ..
chmod 777 -R html
```

## 1.2. Docker基本命令

```sh
controller
compute
master
slave1
server
client
vi /etc/vsftpd/vsftpd.conf
添加anon_root=/opt/

IPADDR=192.1681.200.xx
NETMASK=255.255.255.0
GATEWAY=192.1681.200.1
DNS1=114.114.114.114

[docker]
name=docker
baseurl=
gpgcheck=0
enable=1

systemctl stop firewalld
systemctl disable firewalld

#开启内核转发
vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0


如果还没有 docker group 就添加一个：
sudo groupadd docker
将用户加入该 group 内。然后退出并重新登录就生效啦。
sudo gpasswd -a ${USER} docker
重启 docker 服务
sudo service docker restart
切换当前会话到新 group 或者重启 X 会话
newgrp - docker

#阿里云加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerhub.azk8s.cn",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://nv9ab05p.mirror.aliyuncs.com"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# vi /etc/sysconfig/docker //添加下面两行 IP为server端IP
ADD_REGISTRY='--add-registry 10.0.0.51:5000'
INSECURE_REGISTRY='--insecure-registry 10.0.0.51:5000'
ADD_REGISTRY='--add-registry server:5000'
INSECURE_REGISTRY='--insecure-registry server:5000'

#启动registry仓库
docker run -d -p 5000:5000 --restart=always --name registry docker.io/registry:latest
#启动编排服务rancher
docker run -d -p 8080:8080 --restart=unless-stopped --name rancher  rancher/server:v1.6.5
netstat -auntp |grep 80
yum install -y mariadb mariadb-server bridge-utils lsof 
systemctl enable mariadb
systemctl restart mariadb
mysql_secure_installation

[root@server rancher1.6.5]# docker images --format "table{{.ID}} {{.Repository}}:{{.Tag}}"
主机注册地址
主机连接Rancher API的Base URL是？
 http://192.168.200.20:8080
#删除所有镜像
docker rmi $(docker images -qa)
#删除所有容器
docker rm -f $(docker ps -qa)

docker run -it -p 80:80 --name web server:5000/nginx:latest /bin/bash
-i 
-t tty终端 配合/bin/bash
-d 后台启动
-p 端口物理机
--name 指定容器名
-P 随机映射端口
-p hostPort:containerPort
-p ip:hostPort:containerPort
-p ip::containerPort
-p hostPort:containerPort:udp
-p 8080:8080 -p 443:443 #多个端口映射

#进入容器
docker exec -it web /bin/bash

#容器添加参数  Update configuration of one or more containers
[root@quanlook ~]# docker update --help

Usage:  docker update [OPTIONS] CONTAINER [CONTAINER...]

Update configuration of one or more containers

Aliases:
  docker container update, docker update

Options:
      --blkio-weight uint16        Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --cpu-period int             Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int              Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int          Limit the CPU real-time period in microseconds
      --cpu-rt-runtime int         Limit the CPU real-time runtime in microseconds
  -c, --cpu-shares int             CPU shares (relative weight)
      --cpus decimal               Number of CPUs
      --cpuset-cpus string         CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string         MEMs in which to allow execution (0-3, 0,1)
  -m, --memory bytes               Memory limit
      --memory-reservation bytes   Memory soft limit
      --memory-swap bytes          Swap limit equal to memory plus swap: -1 to enable unlimited swap
      --pids-limit int             Tune container pids limit (set -1 for unlimited)
      --restart string             Restart policy to apply when a container exits
######具体可查看 https://docs.docker.com/engine/reference/commandline/service_update/
# --publish--add  可能存在版本的差别 请大佬指正
docker update \
--restart=always \
--publish--add 81:80 [CONTAINER ID/CONTAINER NAME]

docker container update --publish-rm 8080:808 --publish-add 8080:80 my_container

#Docker是一种流行的容器化平台，通过使用Docker可以有效地构建、部署和管理应用程序。其中，一项常用的功能是修改文件映射，本文将介绍如何进行文件映射的修改。

#在Docker中，文件映射是指将宿主机上的目录映射到Docker容器中的目录，使得容器可以访问宿主机上的文件和目录。文件映射通常在创建容器时进行配置，使用-v参数指定要映射的宿主机目录和容器中的目录，例如：

docker run -v /path/to/host/dir:/path/to/container/dir image
#在运行容器期间，我们可能需要修改已经创建的文件映射。下面是修改步骤：

#停止容器
docker stop container_name
#解除文件映射
docker container update --detach=false --publish-rm /path/to/host/dir container_name
#添加新的文件映射
docker container update --publish-add /new/host/dir:/path/to/container/dir container_name
#重新启动容器
docker start container_name
#上述命令中，我们使用了--detach=false参数使容器保持在停止状态，并使用--publish-rm参数删除旧的文件映射。接着，我们使用--publish-add参数添加新的文件映射。最后，我们使用docker start命令重新启动容器。



#创建(提交)一个镜像
docker commit -a "quanlook@qq.com" -m "修改了/usr/share/nginx/html/" web nginx:v0.1
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
5f515359c7f8        redis               latest
05a60462f8ba        nginx               latest
fe9198c04d62        mongo               3.2
f753707788c5        ubuntu              16.04
f753707788c5        ubuntu              latest
1e0c3dd64ccd        ubuntu              14.04
[root@server ~]# docker images --format "table docker push {{.Repository}}:{{.Tag}}" 
[root@server ~]# docker images --format "table docker push {{.ID}} {{.Repository}}:{{.Tag}}" 


2、三个容器之间使用共享卷
docker run -itd --name web1 -p 81:80 -v /usr/share/nginx/html nginx
docker run -itd --volumes-from web1 --name web2 -p 82:80  nginx
docker run -itd --volumes-from web1 --name web3 -p 83:80 nginx
docker exec -it web1 /bin/bash
echo "I am web1." > /usr/share/nginx/html/index.html

[root@server images]# curl http://10.0.6.126:8080/env/1a5/apps/stacks
[root@server ~]# curl http://10.0.6.127:9093/explore/users

[root@server ~]# mkdir /opt/xiandian
[root@server ~]# docker run -d -it  -P --name xiandian-dir -v /opt/xiandian nginx:latest /bin/bash
e103676b5199ff766cb06b71fcb4c438fc083b4d4e044863db0944370c0fb914
#查询卷
[root@server ~]# docker inspect -f '{{.Config.Volumes}}' xiandian-dir
map[/opt/xiandian:{}]

[root@registry ~]#  curl  127.0.0.1:5000/v2/_catalog

[root@registry ~]# docker run -itd -p 80:80 --name nginx nginx:latest /bin/bash
9dad61c107d56b64319f4c20ffc1e7168a66c35ebb97ff51af1f119f4e621a38
[root@registry ~]# docker ps -a
CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS                         NAMES
9dad61c107d5        nginx:latest                        "/bin/bash"              29 seconds ago      Up 27 seconds       0.0.0.0:80->80/tcp, 443/tcp   nginx
24bcccdd1598        172.30.15.50:5000/registry:latest   "/entrypoint.sh /etc/"   20 hours ago        Up 20 hours         0.0.0.0:5000->5000/tcp        registry
[root@registry ~]# docker exec -it 9dad61c107d5 /bin/bash
root@9dad61c107d5:/# /usr/sbin/nginx
root@9dad61c107d5:/# cd /usr/share/nginx/html
root@9dad61c107d5:/usr/share/nginx/html# rm -rvf index.html 
removed 'index.html'
root@9dad61c107d5:/usr/share/nginx/html# exit
exit
[root@registry ~]# cd /opt
[root@registry opt]# vi index.html 
Welcome to XianDian!
this is container!
Thank you for using nginx.
[root@registry opt]# docker cp /opt/index.html 9dad61c107d5:/usr/share/nginx/html/
[root@registry docker_images]# curl -L 172.30.15.5
Welcome to XianDian!
this is container!
Thank you for using nginx!


centos7 docker容器报 docker Failed to get D-Bus connection 错误
systemctl start nginx
报错内容：Failed to get D-Bus connection: Operation not permitted
报这个错的原因是dbus-daemon没能启动。systemctl并不是不能使用。启动时添加--privileged /usr/sbin/init 即可。docker容器会自动将dbus等服务启动起来。如下：
docker run --privileged -ti --name centos  docker.io/centos:7  /usr/sbin/init

```

## docker私有云

```sh

# owncloud
mkdir  ~/owncloud
docker run -id --name owncloud \
 -p 8081:80  --restart=always \
 -v ~/owncloud:/var/www/html/data   owncloud:latest

# nextcloud:latest
mkdir ~/nextcloud
docker run -id --name nextcloud \
 -p 8080:80 --restart=always  \
 -v ~/nextcloud:/var/www/html/data  nextcloud:latest
```

## Docker快速部署

```sh
#Server节点
HOST_NAME = server
HOST_NAME_NODE = client
HOST_IP =
HOST_IP_NODE =

systemctl  stop firewalld.service
systemctl  disable  firewalld.service
sed -i 's/SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
setenforce 0
sed -i -e 's/#UseDNS yes/UseDNS no/g' -e 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/g' /etc/ssh/sshd_config

if [[ `ip a |grep -w $HOST_IP ` != '' ]];then
        hostnamectl set-hostname $HOST_NAME
elif [[ `ip a |grep -w $HOST_IP_NODE ` != '' ]];then
        hostnamectl set-hostname $HOST_NAME_NODE
else
        hostnamectl set-hostname $HOST_NAME
fi
sed -i -e "/$HOST_NAME/d" -e "/$HOST_NAME_NODE/d" /etc/hosts
echo "$HOST_IP $HOST_NAME" >> /etc/hosts
echo "$HOST_IP_NODE $HOST_NAME_NODE" >> /etc/hosts

systemctl stop iptables
iptables -F
iptables -X
iptables -Z

echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
echo 'net.ipv4.conf.default.rp_filter=0' >> /etc/sysctl.conf
echo 'net.ipv4.conf.all.rp_filter=0' >> /etc/sysctl.conf
sysctl -p

setsebool -P allow_ftpd_full_access on
setsebool -P ftp_home_dir on

mkdir /etc/yum.repos.d/backup
mv /etc/yum.repos.d/{*.repo,backup}
tee /etc/yum.repos.d/local.repo<<EOF
[centos]
name=centos
baseurl=ftp://server/centos
gpgcheck=0
enabled=1
[iaas]
name=iaas
baseurl=ftp://server/iaas/iaas-repo
gpgcheck=0
enabled=1
[pass]
name=paas
baseurl=ftp://server/paas/docker
gpgcheck=0
enabled=1
EOF

yum install -y docker-ce
systemctl restart docker && systemctl enable docker

echo "ADD_REGISTRY='--add-registry $HOST_IP:5000'" >>/etc/sysconfig/docker
echo "INSECURE_REGISTRY='--insecure-registry $HOST_IP:5000'" >>/etc/sysconfig/docker
systemctl daemon-reload 
systemctl restart docker
docker info

# yum install -y haproxy
yum install -y mariadb  mariadb-server bridge-utils
systemctl enable mariadb && systemctl restart mariadb

scp /etc/yum.repos.d/local.repo client:/etc/yum.repos.d/ftp.repo
scp /etc/hosts client:/etc/hosts
scp /etc/sysconfig/docker client:/etc/sysconfig/docker

#Client节点


```

## docker-compose 安装

- GitHub：<https://github.com/docker/compose/releases>

- 镜像加速：<https://ghproxy.com/>

```sh
yum -y install jq curl
# or
apt install -y jq curl
# Github 地址
https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-linux-x86_64

#获取最新版本
repo_releases=(`curl -sS https://api.github.com/repos/docker/compose/releases/latest | jq -r '.tag_name'`)
# 加速下载
curl -L https://ghproxy.com/https://github.com/docker/compose/releases/download/$repo_releases/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 取消进度条可以使用 curl -sS
curl -sSLo /usr/local/bin/docker-composes \ 
https://ghproxy.com/https://github.com/docker/compose/releases/download/$repo_releases/docker-compose-`uname -s`-`uname -m` 

# 添加可执行权限
chmod +x /usr/local/bin/docker-compose
# 查看 docker-compose 版本
docker-compose version
```

**内容持续更新中….**
