---
layout: post
date: 2017-09-16 09:28
title: How to build your own ShadowSocks 
description:  本人自建ShadowSocks过程中参考的方案教程整理而来，涉及Linux服务器端、Windosw、IOS、安卓以及Linux Desktop桌面系统，不同客户端搭建方案不同，愿大家都能翻墙到邻家科学上网，总结如下。
categories: [ShadowSocks,科学上网]
tags: [ShadowSocks]
---
# 一、服务器&客户端介绍
> * Amazon EC2---Centos 6.5
> * iPhone---Wingy/SsrConnectPro
> * android---Shadowsocks
> * windows---Shadowsocks
> * Ubuntu Desktop 16.04 LTS----shadowsocks QT5

## Shadowsocks/Shadowsocks QT5
Shadowsocks都为同一家开源产品，知一而懂所有，入手方便很多。
此文之前的前一夜，自己电脑安装Ubuntu Desktop因访问自由网络受限而苦恼，奋战到凌晨1点终于取得成功。心想不同平台使用ShadowSocks并不能使用户平滑的过度到其他终端而自由科学上网，便有此文总结。

**下载链接：**[Shadowsocks服务器/客户端全家桶][1]
## Amazon EC2
Amazon EC2需选择网络自由国家的linux服务器，由它提供网络代理，其他终端经由它代理流量访问目标网址从而绕过GFW。本人使用的EC2所在国家是日本东京，系统版本为Centos 6.5，服务端网速可达15M，且稳定。

**链接：**[AWS全家桶][2]
## Wingy/SsrConnectPro
Wingy客户端为免费应用，操作界面简洁，易上手，重点是首次配置就成功了，其他也尝试了但未果，遗憾的是9月之后大陆区App Store已经下架该应用，但美区可供下载；欣慰的是，后期找到一款目前在大陆去APP Store还可下载的便是上文的SsrConnectPro，配置方式和Wingy类似，也是不错的。
## windows
windows配置配置最为方便，不多说。

## Ubuntu Desktop 16.04 LTS
在自个老旧的电脑上裸装Ubuntu带桌面系统，在局域网中也可通过windows上Xshell等工具连接，方便实操Linux，Ubuntu是Linux家族中桌面应用众多，如网易云音乐，搜狗输入法等，偏向家用型OS。

# 二、安装教程
## Shadowsocks
通过ShadowsocksR一键安装脚本
![cmd-markdown-logo](https://shadowsocks.be/usr/uploads/shadowsocks.png)
### Linux服务器安装SS
使用root用户登录，运行以下命令：
``` java
wget --no-check-certificate    https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
chmod +x shadowsocksR.sh
./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```
安装完成后，脚本提示如下：
```java
Congratulations, ShadowsocksR server install completed!
Your Server IP        :your_server_ip
Your Server Port      :your_server_port
Your Password         :your_password
Your Protocol         :your_protocol
Your obfs             :your_obfs
Your Encryption Method:your_encryption_method

Welcome to visit:https://shadowsocks.be/9.html
Enjoy it!
```
卸载方式，以root身份运行：
```java
./shadowsocksR.sh uninstall
```
查看SS运行状态：
```java
/etc/init.d/shadowsocks status
```
可以查看 ShadowsocksR 进程是否已经启动。
本脚本安装完成后，已将 ShadowsocksR 自动加入开机自启动。

**使用命令**：
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status

配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocks.log
代码安装目录：/usr/local/shadowsocks

**多用户配置示例：**
```java
{
"server":"0.0.0.0",
"server_ipv6": "[::]",
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":{
    "8989":"password1",
    "8990":"password2"，
    "8991":"password3"
},
"timeout":300,
"method":"aes-256-cfb",
"protocol": "origin",
"protocol_param": "",
"obfs": "plain",
"obfs_param": "",
"redirect": "",
"dns_ipv6": false,
"fast_open": false,
"workers": 1
}

```
## 关于Amazon EC2
 linux服务器云厂商可供自由选择，没有强制选择，唯一的要求便是服务器所在地为国外，若使用EC2建议日本、新加坡优先，其次美国服务器。替代品如godaddy、Vultr等自行甄选。外加一句，Amazon EC2的网速NB，相信没有比他更快的，我在国内的阿里&腾讯的在1/2M左右，比较网速的前提为掏相同的钱才有可比性。AWS **彩蛋**在后面，仔细找。

## Wingy/SsrConnectPro配置
###  Wingy配置
如果你已经安装好Wingy，进入配置，添加ShadowSockets(R)方式，如图：
![cmd-markdown-logo](http://ow5hmv2cu.bkt.clouddn.com/wingy.jpg)

### SsrConnectPro配置
如果你已经安装好SsrConnectPro，进入配置新建就可，没有连接协议选择，配置之后，点击Ping，如果有数据返回表示成功，随后点击Connect即可。如图：
![cmd-markdown-logo](http://ow5hmv2cu.bkt.clouddn.com/wingy.jpg)
## windows配置
![cmd-markdown-logo](http://ow5hmv2cu.bkt.clouddn.com/s-win.jpg)

## Ubuntu Desktop 16.04 LTS配置
Ubuntu客户端配置需要借助浏览器插架：Chrome+SwitchyOmega，FireFox+foxyproxy，我只使用Chrome，配置如下：
### PPA安装
PPA is for Ubuntu >= 14.04.
```java
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```
### shadowsocks-qt5配置
![cmd-markdown-logo](http://ow5hmv2cu.bkt.clouddn.com/ss1.jpg)

![cmd-markdown-logo](http://ow5hmv2cu.bkt.clouddn.com/ss2.jpg)
### SwitchyOmega配置
**下载链接：**[SwitchyOmega ][3]

![SwitchySharp图一](http://ow5hmv2cu.bkt.clouddn.com/swith1.jpg)

![SwitchySharp图二](http://ow5hmv2cu.bkt.clouddn.com/swith2.jpg)

# 三、安装参考教程
为这个世界开源组织和个人表示由衷的感谢，开源精神永垂不朽，光芒四射！感谢技术爱好者和追求自由科学上网的个人将自己的技术经验一字一字码上网络进行分享。
也许部分链接可能失效或者其它原因无法访问，相信本文的图文可以给予最直接的帮助！
彩蛋揭晓：AWS注册绑定信用可，可免费享有一年诸多服务使用权，如S3、EC2等。
> * [ShadowSocks各平台安装教程][4]
> * [linux-ubuntu使用shadowsocks客户端配置][5]
> * [linux配置shadowsocks客户端][6]
> * [各种系统下Shadowsocks客户端的安装与配置][7]


  [1]: https://shadowsocks.org/en/download/clients.html
  [2]: https://amazonaws-china.com/
  [3]: https://github.com/FelisCatus/SwitchyOmega/releases
  [4]: https://shadowsocks.be/9.html
  [5]: http://rrcat.us/?p=250
  [6]: https://my.oschina.net/u/1432769/blog/619651
  [7]: http://www.jeyzhang.com/how-to-install-and-setup-shadowsocks-client-in-different-os.html
