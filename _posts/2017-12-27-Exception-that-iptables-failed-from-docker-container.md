---
layout: post
date: 2017-12-27 16:30
title: Exception that iptables failed from docker container
description: docker容器启动异常 提示iptables failed:iptables --wait -t filter -ADOCKER
categories: [Spark,SparkSQL]
tags: [Spark,SparkSQL]
---

# 问题
自9月初试docker之来，2个月没启动阿里云上的docker，之前运行在上面的Kong因期间服务器重启关闭了，这次启动报错提示如下：
```java
[root@ecs-ali init.d]# docker start kong-database
Error response from daemon: 
driver failed programming external connectivity on endpoint kong-database 
(8f7795a1580942c9c954c7f7e547196a59de5f183626ffc03bc23224dc06ec8f): 
(iptables failed: iptables --wait -t filter -A DOCKER ! 
-i docker0 -o docker0 -p tcp -d 172.18.0.2 --dport 5432 -j ACCEPT: iptables: 
No chain/target/match by that name.
 (exit status 1))
Error: failed to start containers: kong-database
```
看提示以为是防火墙端口原因，执行了以下命令：
```java
/sbin/iptables -I INPUT -p tcp --dport 5432 -j ACCEPT
service iptables save
systemctl restart iptables.service
systemctl status iptables.service
iptables -L -n
```
# 办法
然后再次启动空，发现异常依旧。最终在网上找到解决方案，方法如下：
```java
#我的环境为centos 7,部分命令可能没用

pkill docker 

iptables -t nat -F 

ifconfig docker0 down 

brctl delbr docker0  //# 没有，执行安装yum install bridge-utils

docker -d 

systemctl restart docker 
```
# 参考
[iptables异常][1]


  [1]: http://blog.sina.com.cn/s/blog_8e032fb90102xuon.html
