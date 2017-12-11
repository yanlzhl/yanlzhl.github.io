---
layout: post
date: 2017-10-10 11:00
title: Build hadoop & spark cluster
description: 搭建hadoop&spark集群记录[Build hadoop & spark cluster records]
categories: [Hadoop,Spark,Zookeeper]
tags: [Hadoop,Spark,Zookeeper]
---

# linux设置不同的主机名
此步骤可省略。给每个linux设置不同的主机名方便在集群几点切换时，容易辨认自己所在的服务器以致不易出现失误。我自己在阿里ecs有三个节点如：
```java

 - ecs-ali
 - ecs-ali2
 - ecs-ali3
 
 centos7是个分水岭，设置方法有所不同
=========centos 6.5===========
centos 6.5 : cp /etc/sysconfig/network /etc/sysconfig/network.`date +%Y%m%d.%H%M%S` 备份
[root@ecs-ali3 ~]# cat /etc/sysconfig/network
# Created by anaconda
NETWORKING_IPV6=no
PEERNTP=no
HOSTNAME=ecs-ali3 //修改在此

=========centos 7===========
在centos 7主机名有静态、瞬态或灵活主机名之分，我一并修改为同一个（永久修改主机名，修改静态主机名：）
[root@localhost ~]# hostnamectl set-hostname ecs-ali //xxx为需要设置的主机名
//查看主机名
[root@ecs-ali ~]# hostnamectl --pretty
ecs-ali
[root@ecs-ali ~]# hostnamectl --static
ecs-ali
[root@ecs-ali ~]# hostnamectl --transient
ecs-ali

就像上面展示的那样，在修改静态/瞬态主机名时，任何特殊字符或空白字符会被移除，而提供的参数中的任何大写字母会自动转化为小写。一旦修改了静态主机名，/etc/hostname 将被自动更新。然而，/etc/hosts 不会更新以保存所做的修改，所以你每次在修改主机名后一定要手动更新/etc/hosts，之后再重启CentOS 7。否则系统再启动时会很慢。
```
修改主机名后reboot。

# 修改linux 节点/etc/hosts文件
修改/etc/hosts需要特别注意，在修改之前需了解清楚127.0.0.1/0.0.0.0/静态IP/公网IP使用环境，设置不当，node之间会出现worker注册不上master的问题。

举例：以下是ecs-ali3节点的服务器的配置，其本身设置静态IP主机名映射，同时把原有hosts中的localhost注释掉，**切勿本身映射公网IP**。

我的3台服务器不在同一个内网，分布在华中华北等。阿里云也没有提供技术使其可内网互通，我的搭建过程遇到的坑相对较多，此处多次掉坑。此外，**阿里的服务器需手动配置安全组出入站端口访问权限。**
```java
[root@ecs-ali3 ~]# vim /etc/hosts
47.95.251.123 ecs-ali（公网IP）
101.132.183.123 ecs-ali2（公网IP）
172.19.220.71 ecs-ali3 （静态IP,注意，其它连节点参照此处）
```
三个节点修改之后，相互通过ping命令进行测试。

# 创建用户和用户组
hadoop和spark集群共用同一账户
```java
useradd -s /sbin/nologin spark
usermod -s /bin/bash spark //用于没有权限登陆问题
passwd //修改下密码，可设亦可不设
chown spark:spark /usr/local/spark //变更用户组,确保该目录已经存在
```
**注意：下一步节点之间免密登陆应该用于spark用户，特别是在生产环境。**

# 设置linux节点之间免密登陆
**注意：节点之间免密登陆应该用于spark用户，而非root用户，特别是在生产环境。**

对于需要远程管理其它机器，一般使用远程桌面或者telnet。linux一般只能是telnet。但是telnet的缺点是通信不加密，存在不安全因素，只适合内网访问。为
解决这个问题，推出了通信加密通信协议，即SSH（Secure Shell）。使用非对称加密方式，传输内容使用rsa或者dsa加密，可以避免网络窃听。
hadoop的进程之间同信使用ssh方式，需要每次都要输入密码。为了实现自动化操作，需要配置ssh免密码登陆方式。
配置ssh免密码登录（三个节点m1、s1、s2）
主节点配置：

	1. 首先到用户主目录（cd  ~），ls  -a查看文件，其中一个为“.ssh”，该文件价是存放密钥的。待会我们生成的密钥都会放到这个文件夹中。
	2. 现在执行命令生成密钥： ssh-keygen -t rsa -P ""  (使用rsa加密方式生成密钥)回车后，会提示三次输入信息，我们直接回车即可。（直接回车：一定要使用默认文件名）
	3. 进入文件夹cd  .ssh (进入文件夹后可以执行ls  -a 查看文件) 
	4. 将生成的公钥id_rsa.pub 内容追加到authorized_keys（执行命令：cat id_rsa.pub >> authorized_keys）


从节点配置：

 - 1. 以同样的方式生成秘钥（ssh-keygen -t rsa -P "" ），然后s1和s2将生成的id_rsa.pub公钥追加到m1的authorized_keys中）
 - 2. 在s1中执行命令：scp id_rsa.pub m1:/root/.ssh/id_rsa.pub.s1 ，在s2中执行命令：scp id_rsa.pub m1:/root/.ssh/id_rsa.pub.s2
 - 3. 进入m1执行命令：cat id_rsa.pub.s1 >> authorized_keys ，cat id_rsa.pub.s1 >> authorized_keys
 - 4. 最后将生成的包含三个节点的秘钥的authorized_keys 复制到s1和s2的.ssh目录下（ scp authorized_keys s1:/root/.ssh/， scp authorized_keys s2:/root/.ssh/）

验证ssh免密码登录
	1. 输入命令ssh  localhost(主机名) 根据提示输入“yes” 
	2. 输入命令exit注销（Logout）
	3. 再次输入命令ssh localhost即可直接登录


出了上面以上通过scp命令等方式追加公钥到其他节点也可通过如下命令：
```java
该命令表示，将ecs-ali3的id_rsa.pub直接追加到ecs-ali的authorized_keys文件
[root@ecs-ali3 .ssh]# ssh-copy-id -i ~/.ssh/id_rsa.pub ecs-ali
```
如果是非root用户，如spark用户，需修改authorized_keys的权限为600否则会出现没有权限访问问题
```java
[spark@ecs-ali .ssh]$ chmod 600 authorized_keys

-rw------- 1 spark spark 1187 Dec  1 19:28 authorized_keys //600
```

	
# 安装JDK
```java
创建JDK安装目录
    mkdir /usr/local/java

下载JDk
    /usr/local/java
    
    wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/server-jre-8u131-linux-x64.tar.gz

解压
    tar -zxvf server-jre-8u131-linux-x64.tar.gz

设置JAVA环境变量
    vim /etc/profile
    
    #Java Environment Configuration
    export JAVA_HOME=/usr/local/java/jdk1.8.0_131
    export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
    export CLASSPATH=.:/$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
    
    source /etc/profile

查看安装正确与否
[root@ecs-ali3 ~]# java -version
    java version "1.8.0_131"
    Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)

```


# 安装配置Hadoop
## 安装hadoop
从Apache官方下载速度龟速，建议从国内镜像站下载
```java
创建hadoop目录
    mkdir /usr/local/hadoop
    cd /usr/local/hadoop

清华大学镜像站
    https://mirrors4.tuna.tsinghua.edu.cn/
    
    wget https://mirrors4.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoo     p-2.8.1/hadoop-2.8.1.tar.gz

配置环境变量
    export HADOOP_HOME=/usr/local/hadoop/hadoop-2.8.1
    export PATH=$PATH:$HADOOP_HOME/bin
```
## 配置hadoop文件
本人看到网络上hadoop集群配置的方式是修改master节点的hadoop文件，然后将整个hadoop打包（上百兆，那么大）scp到各个子节点，觉得有点慢，就是有点慢！我采用一个一个修改个节点上的配置文件。我的做法前提是，能确保各个节点上的hadoop配置文件内容是一样的。这也是大家都用scp到各节点的原因，100%保证各节点配置相同。
**强调：配置文件所有节点务必保持一致！
强调：配置文件所有节点务必保持一致！
强调：配置文件所有节点务必保持一致！**

**配置文件路径：/usr/local/hadoop-2.8.1/etc/hadoop** 
配置文件几乎都提供了模板文件，再此内容上追加以下内容
以下ecs-ali是我集群的master节点

### hadoop-env.sh
```java
添加JDK支持：
       export JAVA_HOME=/usr/local/java/jdk1.8.0_131
```
### core-site.sh
```java
注意：必须加在<configuration></configuration>节点内
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/hadoop-2.6.0/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://ecs-ali:9000</value>
    </property>
</configuration>

```
### hdfs-site.xml
```java
<property>
    <name>dfs.name.dir</name>
    <value>/home/hadoop/hadoop-2.6.0/dfs/name</value>
    <description>Path on the local filesystem where the NameNode stores the namespace and transactions logs persistently.</description>
</property>
 
<property>
    <name>dfs.data.dir</name>
    <value>/home/hadoop/hadoop-2.6.0/dfs/data</value>
    <description>Comma separated list of paths on the local filesystem of a DataNode where it should store its blocks.</description>
</property>
<property>
    <name>dfs.replication</name>
    <value>2</value> //一主两从
</property>
<property>
    <name>dfs.permissions </name>
    <value>false</value> //spark访问方便，因为是spark用户，缺少权限
</property>

 - 列表项

```

### mapred-site.xml
```java
<property> 
   <name>mapred.job.tracker</name> 
   <value>ecs-ali:9001</value> 
   <description>Host or IP and port of JobTracker.</description>
</property>

```

### mapred-env.sh
```java
export JAVA_HOME=export JAVA_HOME=/usr/local/java/jdk1.8.0_131
```

### slaves
```java
创建slaves
touch slaves
vim slaves

[spark@ecs-ali hadoop]$ cat slaves 
ecs-ali2
ecs-ali3
ecs-ali
```

### master
```java
创建master文件
touch master
vim master

[spark@ecs-ali hadoop]$ cat master
ecs-ali
```

#启动Hadoop集群
## 格式化HDFS文件系统的namenode、datanode
```java
cd hadoop-2.8.1  //进入hadoop-2.8.1目录
bin/hdfs namenode -format  //格式化
bin/hdfs datanode -format 
```

## 启动/停止hadoop集群
```java
sbin/start-all.sh //开启进程，主要是启动hdfs、yarn
sbin/stop-all.sh

[root@ecs-ali hadoop-2.8.1]# sbin/start-all.sh
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [ecs-ali]
ecs-ali: starting namenode, logging to /usr/local/hadoop-2.8.1/logs/hadoop-root-namenode-ecs-ali.out
ecs-ali: starting datanode, logging to /usr/local/hadoop-2.8.1/logs/hadoop-root-datanode-ecs-ali.out
ecs-ali2: starting datanode, logging to /usr/local/hadoop-2.8.1/logs/hadoop-root-datanode-ecs-ali2.out
ecs-ali3: starting datanode, logging to /usr/local/hadoop-2.8.1/logs/hadoop-root-datanode-ecs-ali3.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.8.1/logs/hadoop-root-secondarynamenode-ecs-ali.out
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop-2.8.1/logs/yarn-root-resourcemanager-ecs-ali.out
ecs-ali3: starting nodemanager, logging to /usr/local/hadoop-2.8.1/logs/yarn-root-nodemanager-ecs-ali3.out
ecs-ali2: starting nodemanager, logging to /usr/local/hadoop-2.8.1/logs/yarn-root-nodemanager-ecs-ali2.out
ecs-ali: starting nodemanager, logging to /usr/local/hadoop-2.8.1/logs/yarn-root-nodemanager-ecs-ali.out

```

## 验证
```java
[root@ecs-ali hadoop-2.8.1]# jps //jps命令
19616 Jps
18995 DataNode
19427 NodeManager
18858 NameNode
19163 SecondaryNameNode

hadoop dfsadmin -report //
[root@ecs-ali hadoop-2.8.1]# hadoop dfsadmin -report //hadoop命令
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

Configured Capacity: 126418354176 (117.74 GB)
Present Capacity: 92764831744 (86.39 GB)
DFS Remaining: 92764733440 (86.39 GB)
DFS Used: 98304 (96 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0
Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (3):

Name: 101.132.181.208:50010 (ecs-ali3)
Hostname: 127.0.0.1
Decommission Status : Normal
Configured Capacity: 42139451392 (39.25 GB)
DFS Used: 32768 (32 KB)
Non DFS Used: 5154770944 (4.80 GB)
DFS Remaining: 34820493312 (32.43 GB)
DFS Used%: 0.00%
DFS Remaining%: 82.63%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Dec 11 18:13:24 CST 2017


Name: 101.132.183.227:50010 (ecs-ali2)
Hostname: 127.0.0.1
Decommission Status : Normal
Configured Capacity: 42139451392 (39.25 GB)
DFS Used: 32768 (32 KB)
Non DFS Used: 5400231936 (5.03 GB)
DFS Remaining: 34575032320 (32.20 GB)
DFS Used%: 0.00%
DFS Remaining%: 82.05%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Dec 11 18:13:26 CST 2017


Name: 172.17.108.179:50010 (localhost)
Hostname: 127.0.0.1
Decommission Status : Normal
Configured Capacity: 42139451392 (39.25 GB)
DFS Used: 32768 (32 KB)
Non DFS Used: 16606056448 (15.47 GB)
DFS Remaining: 23369207808 (21.76 GB)
DFS Used%: 0.00%
DFS Remaining%: 55.46%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Dec 11 18:13:24 CST 2017

亦可通过web-ui查看
hdfs 管理界面，查看文件系统
http://master:50070/dfshealth.html#tab-overview //master公网IP

yarn 管理界面，查看job运行情况
http://master:8088/cluster //job运行时可查看
```
# hadoop wordcount
在/usr/local/hadoop-2.8.1/下创建helloHadoop.txt文件
```java
[root@ecs-ali hadoop-2.8.1]# cat helloHadoop.txt 
Hello hadoop
hello spark
hello bigdata

hadoop fs -mkdir -p /Hadoop/Input
hadoop fs -put wordcount.txt /Hadoop/Input
hadoop jar /usr/local/hadoop-2.8.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar wordcount /Hadoop/Input /Hadoop/Output //默认存在


[root@ecs-ali mapreduce]# hadoop fs -cat /Hadoop/Output/*
Hello	1
bigdata	1
hadoop	1
hello	2
spark	1

```

# zookeeper安装

##下载
```java
mkdir /usr/local/zookeeper
cd /usr/local/zookeeper
还是从清华镜像站下载
https://mirrors4.tuna.tsinghua.edu.cn/

wget https://mirrors4.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
```
## 配置hosts文件
如果已经安装好hadoop集群，即/etc/hosts文件修改好各node IP主机名映射如下所示：
```java
[root@ecs-ali local]# cat /etc/hosts
101.132.183.227 ecs-ali2
101.132.181.208 ecs-ali3
172.17.108.179 ecs-ali
```
## 配置zookeeper文件
```java
使用zoo.cfg文件
cd /usr/local/zookeeper-3.4.10/conf
cp zoo_sample.cfg zoo.cfg

配置节点地址
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper-3.4.10/data
dataLogDir=/usr/local/zookeeper-3.4.10/logs
clientPort=2181
server.0=172.17.108.179:2888:3888
server.1=ecs-ali2:2888:3888
server.2=ecs-ali3:2888:3888

远程分发此文件,以ecs-ai2为例 ecs-ali3也需要一份
[root@ecs-ali conf]# scp zoo.cfg root@ecs-ali2:/usr/local/zookeeper-3.4.10/conf
zoo.cfg                                     100% 1074    40.7KB/s   00:00

创建myid
在zoo.cfg中dataDir配置的路径下创建myid,我的是zookeeper-3.4.10/data
我们配置的dataDir指定的目录下面，创建一个myid文件，里面内容为一个数字，用来标识当前主机，conf/zoo.cfg文件中配置的server.X中X为什么数字，则myid文件中就输入这个数字，例如：

[root@ecs-ali data]# echo "0" >> myid
[root@ecs-ali data]# cat myid
0
[root@ecs-ali2 data]# echo "1" >> myid
[root@ecs-ali3 data]# echo "2" >> myid


启动zk
[root@ecs-ali zookeeper-3.4.10]# bin/zkServer.sh start

查看各节点状态
[root@ecs-ali zookeeper-3.4.10]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: leader 

[root@ecs-ali2 zookeeper-3.4.10]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower

[root@ecs-ali3 zookeeper-3.4.10]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower
```

# spark集群
hadoop集群如果已经搭建好，再配置spark相对容易许多，spark集群可以独立于hadoop亦可基于hadoop集群运行。
##下载
```java
mkdir /usr/local/spark
cd /usr/local/spark
还是从清华镜像站下载
https://mirrors4.tuna.tsinghua.edu.cn/

wget https://mirrors4.tuna.tsinghua.edu.cn/apache/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.7.tgz

配置环境变量
export SCALA_HOME=/usr/local/scala/scala-2.11.11
export PATH=$PATH:$SCALA_HOME/bin

source /etc/profile
```

##配置spark文件
spark集群同hadoop集群一样，配置文件务必保持一致，可采用在master上配置好然后使用scp命令分发到子节点。
```java
//配置spark-env.sh
cd /usr/local/spark/spark-2.2.0-bin-hadoop2.7/conf
cp spark-env.sh.template spark-env.sh 
vim spark-env.sh 
#export SPARK_MASTER_IP=ecs-ali //使用spark-standalone不用zk时绑定master IP
export JAVA_HOME=/usr/local/java/jdk1.8.0_131
export HADOOP_CONF_DIR=/usr/local/hadoop-2.8.1/etc/hadoop
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=ecs-ali:2181,ecs-ali2:2181,ecs-ali3:2181 -Dspark.deploy.zookeeper.dir=/spark"  //通过zk实现spark高可用

修改slaves文件
cp slaves.template slaves
vim slaves
写入子节点名称到slaves文件
ecs-ali2
ecs-ali3

```
## 启动spark
```java
sbin/start-all.sh //启动集群

[spark@ecs-ali2 spark-2.2.0-bin-hadoop2.7]$ sbin/start-all.sh 
starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/spark-2.2.0-bin-hadoop2.7/logs/spark-spark-org.apache.spark.deploy.master.Master-1-ecs-ali2.out
ecs-ali3: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/spark-2.2.0-bin-hadoop2.7/logs/spark-spark-org.apache.spark.deploy.worker.Worker-1-ecs-ali3.out
ecs-ali2: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/spark-2.2.0-bin-hadoop2.7/logs/spark-spark-org.apache.spark.deploy.worker.Worker-1-ecs-ali2.out
```

## spark wordcount
在hadoop 搭建后put了helloHadoop.txt文件到Hadoop/Input
下面通过spark读取hdfs中该文件
```java
运行spark-shell.sh

scala> val file=sc.textFile("hdfs://ecs-ali:9000/Hadoop/Input/helloHadoop.txt")
file: org.apache.spark.rdd.RDD[String] = hdfs://ecs-ali:9000/Hadoop/Input/helloHadoop.txt MapPartitionsRDD[5] at textFile at <console>:24

scala> val rdd = file.flatMap(line => line.split(" ")).map(word => (word,1)).reduceByKey(_+_)
rdd: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[8] at reduceByKey at <console>:26

scala> rdd.collect()
res1: Array[(String, Int)] = Array((spark,1), (Hello,1), (hadoop,1), (hello,2), (bigdata,1))

scala> rdd.foreach(println)
(spark,1)
(Hello,1)
(hadoop,1)
(hello,2)
(bigdata,1)
```

#参考
- [spark集成hadoop][1]
- [zookeeper集群][2]


  [1]: https://www.cnblogs.com/purstar/p/6293605.html
  [2]: http://blog.csdn.net/shirdrn/article/details/7183503/
