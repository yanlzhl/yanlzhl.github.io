---
layout: post
date: 2017-09-19 17:00
title: Build Docker environment base on Centos 7 
description: 在centos 7上搭建Docker环境
categories: [Docker,Centos]
tags: [Docker,Centos]
---
# 系统要求
> Docker CE 支持 *64 位版本 CentOS 7*，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 overlay2 存储层驱动）无法使用，并且部分功能可能不太稳定。

# 卸载旧版本
旧版本的 Docker 称为 docker 或者 docker-engine，使用以下命令*卸载旧版本*：
```java
sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```
#使用 yum 源 安装
执行以下命令*安装依赖包*：
```java
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
***鉴于国内网络问题，强烈建议使用国内源，下面先介绍国内源的使用。***

## 国内源
执行下面的命令添加 yum 软件源：
```java
sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

# 安装 Docker CE

更新 yum 软件源缓存，并安装 docker-ce。
```java
sudo yum makecache fast
sudo yum install docker-ce
```

# 启动 Docker CE
```java
sudo systemctl enable docker
sudo systemctl start docker
```
# 建立 docker 用户组
> 默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要***使用 docker 的用户加入 docker 用户组。***
 
## 建立 docker 组：
```java
sudo groupadd docker
```
## 将当前用户加入 docker 组：
```java
sudo usermod -aG docker $USER
```

# 镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置,点击此处进入[阿里云加速器官网][1]，配置个人专属加速器地址
```java
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://XXXXXXX.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
# 参考文档

> [Docker入门到实践][2]


  [1]: https://cr.console.aliyun.com/
  [2]: https://yeasy.gitbooks.io/docker_practice/content/install/centos.html
