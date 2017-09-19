---
layout: post
date: 2017-09-19 19:00
title: Build Kong environment base on Docker 
description: 负责公司内部项目建设，给运维团队使用的技术做二次整合开发，其中API Gataway使用开源的Kong，感觉挺有意思，研究一二；上篇文章写道“”在centos 7上搭建Docker环境”，因此本文将在Docker的基础上构建Kong.
categories: [Docker,Centos]
tags: [Docker,Centos]
---
> Kong官方的存储数据容器为cassandra & postgresql.在使用cassandra时一直没有成功，后续实践有结果后在完善此文，下面给出基于postgresql的搭建方法。

# Docker 命令解释
以下是使用Kong使用到的命令
```java
docker run 运行docker镜像
-d:     后台运行容器，并返回容器ID；
-i:     以交互模式运行容器，通常与 -t 同时使用；
-t:     为容器重新分配一个伪输入终端，通常与 -i 同时使用；
-it:    -i -t的组合
--name="kong-database"/--name kong-database:  为容器指定一个名称；
-e:     "POSTGRES_USER=kong": 设置环境变量；
--link=[]: 添加链接到另一个容器；
```
参见：[菜鸟教程Docker命令大全][1]

# Pull postgresql镜像
```java
docker run -d --name kong-database \
              -p 5432:5432 \
              -e "POSTGRES_USER=kong" \
              -e "POSTGRES_DB=kong" \
              postgres:9.4
```
# 将Kong数据迁移配置到postgresql
```java
docker run -it --rm \
    --link kong-database:kong-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong:latest kong migrations up
```

# 启动Kong容器
```java
docker run -d --name kong \
    --link kong-database:kong-database \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -p 8000:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    kong:latest
```
# 参考资料
> [Kong官网][2]  
> [菜鸟教程Docker命令大全][3]


  [1]: http://www.runoob.com/docker/docker-command-manual.html
  [2]: https://getkong.org/install/docker/
  [3]: http://www.runoob.com/docker/docker-command-manual.html
