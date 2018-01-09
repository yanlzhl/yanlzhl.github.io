---
layout: post
date: 2018-01-09 15:19
title: Build Kubernetes[k8s] cluster
description:  k8s集群搭建过程记录以及碰到的问题并给出解决方案。
categories: [Kubernetes]
tags: [Kubernetes,Docker]
---


# 准备工作
我是先对docker初步到进阶的了解学习，常用的命令，基本的容器构建和docker原理概念学习完之后，步入k8s。即使这样，刚看k8s官方文档时，其设计理念、组件构成以及生态需要了解掌握的知识不在少数。为此，花了几天时间在官方文档&个人博客上做了上述方面初步了解，较为直观的便可分为两方面：

**master:**

 - API server
 - Controller Manager
 - Scheduler
 - Etcd


**node:**
    
 - kublet
 - kube-proxy
 - pod


 
 

**Kubernetes集群组件:**

 - **etcd** 一个高可用的K/V键值对存储和服务发现系统
 - **flannel** 实现夸主机的容器网络的通信
 - **kube-apiserver** 提供kubernetes集群的API调用
 - **kube-controller-manager** 确保集群服务
 - **kube-scheduler** 调度容器，分配到Node
 - **kubelet** 在Node节点上按照配置文件中定义的容器规格启动容器
 - **kube-proxy** 提供网络代理服务

这些是在安装过程中直接接触的组件，了解其用途和原理会更胸有成竹吧。
若没有docker基础，应优先学习docker，在构建k8s时更易切解决碰到的问题。
# 搭建过程
官网给出的搭建流程不是很满意，会出问题，环境不同异常不同，排查起来需要用时间填坑；此外也有大量开发者给出的个人经验很有借鉴意义但也是有漏洞，或不详尽，或漏洞百出，需要读多篇这样的文章，相互比较找出漏洞形成完整的搭建思路才方便一部搭建到位，说简单也简单，复杂也是。

----------
- 环境：centos 7
- ecs-ali2  master
- ecs-ali3  node

因另外一台服务器负载较高，所以就搭建2个节点的集群，还是有别于但节点服务器的。3+节点的单间过程和此文相同，复制node搭建过程便可。

----------
**注意：如果你的服务器已经安装过docker，依据我的建议是应该卸载干净，不然会因kubernetes-node和docker产生冲突，导致node启动不了，找不到服务。**

## master环境搭建
### 关闭防火墙
我因使用的是iptables,在以前安装centos7时就变更了防火墙软件，不然建议先禁用centos7自带的Firewall
```python
systemctl stop firewalld   master node环境各执行一遍
systemctl disable firewalld
```
### 确保epel源
```python
yum -y install epel-release  master node环境各执行一遍
```
### master安装etcd kubernetes-master
```python 
yum -y install etcd kubernetes-master
```

### 编辑/etc/etcd/etcd.conf文件
```python
###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"

# The port on the local server to listen on.
KUBE_API_PORT="--port=8080"

# Port minions listen on
KUBELET_PORT="--kubelet-port=10250"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# default admission control policies
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"

# Add your own!
KUBE_API_ARGS=""
[root@ecs-ali2 kubernetes]# cat /etc/etcd/etcd.conf
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"     //※在此
#ETCD_WAL_DIR=""
#ETCD_LISTEN_PEER_URLS="http://localhost:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"   //※在此
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="default"                             //※在此
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#
#[Clustering]
#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://ecs-ali2:2379"     //※在此
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
#ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[Security]
#ETCD_CERT_FILE=""
#ETCD_KEY_FILE=""
#ETCD_CLIENT_CERT_AUTH="false"
#ETCD_TRUSTED_CA_FILE=""
#ETCD_AUTO_TLS="false"
#ETCD_PEER_CERT_FILE=""
#ETCD_PEER_KEY_FILE=""
#ETCD_PEER_CLIENT_CERT_AUTH="false"
#ETCD_PEER_TRUSTED_CA_FILE=""
#ETCD_PEER_AUTO_TLS="false"
#
#[Logging]
#ETCD_DEBUG="false"
#ETCD_LOG_PACKAGE_LEVELS=""
#ETCD_LOG_OUTPUT="default"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#ETCD_AUTO_COMPACTION_RETENTION="0"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#ETCD_METRICS="basic"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple"
```

### 编辑/etc/kubernetes/apiserver文件
```python
###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"

# The port on the local server to listen on.
KUBE_API_PORT="--port=8080"

# Port minions listen on
KUBELET_PORT="--kubelet-port=10250"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# default admission control policies
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"

# Add your own!
KUBE_API_ARGS=""
```

### 启动etcd、kube-apiserver、kube-controller-manager、kube-scheduler等服务，并设置开机启动。
```python
建议写一个脚本 kubernetes-master-start.sh，每次执行脚本就好

#!/usr/bin/env bash  
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
systemctl restart $SERVICES
systemctl enable $SERVICES
systemctl status $SERVICES
done

保存退出后执行下面命令增加可执行权限：
chmod +x kubernetes-master-start.sh
```

### etcd、kube-apiserver、kube-controller-manager、kube-scheduler等服务停止脚本
```python
建议写一个脚本 kubernetes-master-end.sh，每次执行脚本就好

#!/usr/bin/env bash

for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
systemctl stop $SERVICES
systemctl status $SERVICES
done

保存退出后执行下面命令增加可执行权限：
chmod +x kubernetes-master-start.sh
```
执行后，service的status将会是active (running)，否则将是空的，或者执行：
```python
netstat -ntlp
若有：
tcp        0      0 172.19.17.95:2380       0.0.0.0:*               LISTEN      25633/etcd
tcp6       0      0 :::10251                :::*                    LISTEN      25728/kube-schedule 
tcp6       0      0 :::6443                 :::*                    LISTEN      25667/kube-apiserve 
tcp6       0      0 :::2379                 :::*                    LISTEN      25633/etcd          
tcp6       0      0 :::10252                :::*                    LISTEN      25696/kube-controll 
tcp6       0      0 :::8080                 :::*                    LISTEN      25667/kube-apiserve

则恭喜你成了
```

### 在etcd中定义flannel网络
```python
etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}
```

然后你可以访问master节点8080端口，将会返会json格式的节点环境数据！


## node环境搭建
**执行下面命令前，确认是否已经安装docker，若是则卸载，不然出问题，即使卸载后，kubernetes-node也会再次自动装上docker。**
### 安装flannel和kubernetes-node
```python
yum -y install flannel kubernetes-node
```
### 编辑/etc/etcd/etcd.conf文件
```python
# Flanneld configuration options  

# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD_ENDPOINTS="http://ecs-ali2:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_PREFIX="/atomic.io/network"

# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
```

### 修改/etc/kubernetes/config文件
```python
###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://ecs-ali2:8080"

```

### 在所有Node节点上启动kube-proxy,kubelet,docker,flanneld等服务，并设置开机启动
```python
建议写一个脚本 kubernetes-node-start.sh，每次执行脚本就好

#!/usr/bin/env bash

for SERVICES in kube-proxy kubelet docker; do
systemctl restart $SERVICES
systemctl enable $SERVICES
systemctl status $SERVICES
done

保存退出后执行下面命令增加可执行权限：
chmod +x kubernetes-master-start.sh
```

### 结束kube-proxy,kubelet,docker,flanneld等服务
```python
建议写一个脚本 kubernetes-node-start.sh，每次执行脚本就好

#!/usr/bin/env bash
for SERVICES in kube-proxy kubelet docker; do
systemctl stop $SERVICES
systemctl status $SERVICES
done

保存退出后执行下面命令增加可执行权限：
chmod +x kubernetes-master-start.sh

```

执行后，service的status将会是active (running)，否则将是空的，或者执行：
```python
netstat -ntlp
若有：
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      14484/kubelet       
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      14080/kube-proxy
tcp6       0      0 :::10250                :::*                    LISTEN      14484/kubelet       
tcp6       0      0 :::10255                :::*                    LISTEN      14484/kubelet       
tcp6       0      0 :::4194                 :::*                    LISTEN      14484/kubelet

则恭喜你成了
```

### 集群环境验证
在master上执行如下命令
```python
kubectl get node

我的是：
[root@ecs-ali2 yum.repos.d]# kubectl get node
NAME       STATUS    AGE
ecs-ali3   Ready     1h

状态为Ready，则说明集群搭建成功
```

# 官方文档采坑点
https://www.kubernetes.org.cn/doc-16，若是参考这个，以下是一些碰到的问题：
在添加添加virt7-testing源，其使用下面信息添加源：
```python
[virt7-testing]
name=virt7-testing
baseurl=http://cbs.centos.org/repos/virt7-testing/x86_64/os/
gpgcheck=0
```
然后安转过程提示：
```python
failure: repodata/repomd.xml from virt7-testing: [Errno 256] No more mirrors to try.
http://cbs.centos.org/repos/virt7-testing/x86_64/os/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
```
解决办法：
替换baseurl
```python
baseurl=http://cbs.centos.org/repos/virt7-docker-common-candidate/x86_64/os/
```
反正照着官方汉化文档我没有成功，应该不是我智商着急的原因！

# 参考文档
- [k8s学习（强烈推荐，写的不错）][1]
- [k8s中文官方文档][2]
- [k8s集群搭建][3]


  [1]: http://qinghua.github.io/kubernetes-in-mesos-1/
  [2]: https://www.kubernetes.org.cn/doc-16
  [3]: http://blog.csdn.net/magerguo/article/details/72123259?locationNum=3&fps=1
