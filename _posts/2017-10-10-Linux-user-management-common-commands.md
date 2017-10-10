---
layout: post
date: 2017-10-10 11:00
title: Linux 用户管理命令
description: Linux 用户管理命令。学了不一定代表一直能掌握，不使用一定会忘记。
categories: [Linux]
tags: [Linux]
---

> 用户账号的管理工作主要涉及到用户账号的添加、修改和删除。
添加用户账号就是在系统中创建一个新账号，然后为新账号分配用户号、用户组、主目录和登录Shell等资源。

# Linux系统用户账号
## useradd命令
> **添加用户账号**就是在系统中创建一个新账号，然后为新账号分配用户号、用户组、主目录和登录Shell等资源。刚添加的账号是被锁定的，无法使用。
useradd 选项 用户名
选项:
-c comment 指定一段注释性描述。
-d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
-g 用户组 指定用户所属的用户组。
-G 用户组，用户组 指定用户所属的附加组。
-s Shell文件 指定用户的登录Shell。
-u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。
```java
# useradd –d /usr/sam -m sam

此命令创建了一个用户sam，
其中-d和-m选项用来为登录名sam产生一个主目录/usr/sam（/usr为默认的用户主目录所在的父目录）。

# useradd -s /bin/sh -g group –G adm,root gem

此命令新建了一个用户gem，该用户的登录Shell是/bin/sh，它属于group用户组，同时又属于adm和root用户组，其中group用户组是其主组。
这里可能新建组：#groupadd group及groupadd adm　
增加用户账号就是在/etc/passwd文件中为新用户增加一条记录，同时更新其他系统文件如/etc/shadow, /etc/group等。
```
## userdel命令
> userdel 选项 用户名
常用的选项是-r，它的作用是把用户的主目录一起删除。

``` java
# userdel sam

此命令删除用户sam在系统文件中（主要是/etc/passwd, /etc/shadow, /etc/group等）的记录，同时删除用户的主目录。
```
## usermod命令
> **修改用户账号**就是根据实际情况更改用户的有关属性，如用户号、主目录、用户组、登录Shell等。
usermod 选项 用户名
常用的选项包括-c, -d, -m, -g, -G, -s, -u以及-o等，这些选项的意义与useradd命令中的选项一样，可以为用户指定新的资源值。

``` java
# usermod -s /bin/ksh -d /home/z –g developer sam

此命令将用户sam的登录Shell修改为ksh，主目录改为/home/z，用户组改为developer。
```
## passwd命令 
> 用户管理的一项重要内容是用户口令的管理。用户账号刚创建时没有口令，但是被系统锁定，无法使用，必须为其指定口令后才可以使用，即使是指定空口令。

>指定和修改用户口令的Shell命令是passwd。超级用户可以为自己和其他用户指定口令，普通用户只能用它修改自己的口令。
passwd 选项 用户名
可使用的选项：
代码:
-l 锁定口令，即禁用账号。
-u 口令解锁。
-d 使账号无口令。
-f 强迫用户下次登录时修改口令。

``` java
如果默认用户名，则修改当前用户的口令。
例如，假设当前用户是sam，则下面的命令修改该用户自己的口令：
$ passwd
Old password:******
New password:*******
Re-enter new password:*******

如果是超级用户，可以用下列形式指定任何用户的口令：
# passwd sam
New password:*******
Re-enter new password:*******
普通用户修改自己的口令时，passwd命令会先询问原口令，验证后再要求用户输入两遍新口令，如果两次输入的口令一致，则将这个口令指定给用户；而超级用户为用户指定口令时，就不需要知道原口令。

# passwd -d sam
此命令将用户sam的口令删除，这样用户sam下一次登录时，系统就不再询问口令。

passwd命令还可以用-l(lock)选项锁定某一用户，使其不能登录
# passwd -l sam

新建用户异常：
useradd -d /usr/hadoop -u 586 -m hadoop -g hadoop

1 Creating mailbox file: 文件已存在 
删除即可 rm -rf /var/spool/mail/用户名

2 useradd: invalid numeric argument 'hadoop'
这是由于hadoop组不存在 请先建hadoop组
通过cat /etc/passwd 可以查看用户的pass
cat /etc/shadow 可以查看用户名
cat /etc/group 可以查看 组
```

# Linux系统用户组

## groupadd命令
> groupadd 选项 用户组
选项：
-g GID 指定新用户组的组标识号（GID）。
-o 一般与-g选项同时使用，表示新用户组的GID可以与系统已有用户组的GID相同。


**命令操作**
``` java
# groupadd group1

此命令向系统中增加了一个新组group1，新组的组标识号是在当前已有的最大组标识号的基础上加1。

#groupadd -g 101 group2

此命令向系统中增加了一个新组group2，同时指定新组的组标识号是101。
```

## groupdel命令
> **删除一个已有的用户组**

**命令操作**
``` java
#groupdel group1

此命令从系统中删除组group1。
```
## groupmod命令
> **修改用户组的属性**
groupmod 选项 用户组
选项：
-g GID 为用户组指定新的组标识号。
-o 与-g选项同时使用，用户组的新GID可以与系统已有用户组的GID相同。
-n 新用户组 将用户组的名字改为新名字


**命令操作**
```java
# groupmod -g 102 group2

此命令将组group2的组标识号修改为102。

# groupmod –g 10000 -n group3 group2

此命令将组group2的标识号改为10000，组名修改为group3。
```

## newgrp 
```java
newgrp root

这条命令将当前用户切换到root用户组，前提条件是root用户组确实是该用户的主组或附加组。类似于用户账号的管理，用户组的管理也可以通过集成的系统管理工具来完成。
```
# chmod chown chgrp
## chmod
> 文件属性的设置方式有两种，数字和标记。
``` java
说来话长...下回分解
```

## chown
> chown(转变文件拥有者) change owner
A: chown 用户名 文件/目次
B：chown 用户名：用户组：文件/目录
选项
-R 选项意味着对所有子目录下的文件也都进行同样的操作
-h 选项意味着在改变符号链接文件的属主时不影响该链接所指向的目标文件.

```java
chown -R root /etc/config.cfg

 A：chown -R -h 用户名 文件/目录
注意：一旦将文件的所有权交给了另一个用户，就无法再重新收回它的所有权，最终只能求助于系统管理员.

chown -R root:root /home

B：chown 用户名：用户组：文件/目录
若是整个目录下的都改，则加-R参数用于递归。
```

## chgrp
> chgrp(转变文件所属用户组) change group
chgrp 选项 目录/文件
选项
-R 参数用于递归

```java
chgrp -R user smb.conf
```
# 参考
> * [linux 新建用户、用户组 以及为新用户分配权限][1]
> * [Linux系统chmod,chown和chgrp的区别][2]


  [1]: http://www.cnblogs.com/clicli/p/5943788.html
  [2]: http://blog.csdn.net/majishushu/article/details/54619726
