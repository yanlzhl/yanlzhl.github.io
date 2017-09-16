---
layout: post
date: 2017-09-16 14:10
title: How to solve the problem of SSH refuse connection
description: 物理机上安装Linux Mint以及Ubuntu时，远程连接该主机均出现“ssh服务器拒绝了密码,请再试一次”。
categories: [Linux,SSH]
tags: [SSH,Linux]
---
如果安装使用Centos 7，若要使用诸如Xshell等SSH工具远程连接，可轻松接入；而使用Linux Mint或者ubantu系统，因系统为提供该软件服务，则需要安装配置SSH且开启**需要用初装的用户账号给root设置管理密码**

1.查看是否已安装
```java
ps -e|grep ssh
```
2.安装SSH系列软件
```java
sudo apt-get install openssh-server
sudo apt-get install openssh-client
```
3.启动SSH并查看状态
```java
/etc/init.d/ssh start
ps -e|grep ssh

root@yanlz-linux:~# ps -e|grep ssh
 2188 ?        00:00:00 sshd  //表示启动成功
```
4.编辑SSH配置文件
```java
 vim /etc/ssh/sshd_config
 
# Authentication:
LoginGraceTime 120
PermitRootLogin without passwd //此处略有不同
StrictModes yes
改成
#Authentication:
LoginGraceTime 120
PermitRootLogin yes
StrictModes yes
```
5.因系统时初次安装使用，加入root的密码在安装的时候为admin,那么此时若要使用SSH，仍然需要初始化密码，否则远程XShell等工具连接仍然会出现“ssh服务器拒绝了密码,请再试一次”,假如此次密码保持不变，输入仍为admin,则在XShell连接时使用root/admin连接即可：
```java
root@yanlz-linux:~# sudo passwd root //用sudo修改root帐户Password

root@yanlz-linux:~# Enter new UNIX password: 
root@yanlz-linux:~# Retype new UNIX password:
```

