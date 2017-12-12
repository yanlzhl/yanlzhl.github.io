---
layout: post
date: 2017-12-12 16:00
title: Spark learning summary
description: spark核心概念理解
categories: [Hadoop,Spark]
tags: [Hadoop,Spark]
---

# 何为spark
Apache Spark is a *fast* and *general-purpose* cluster computing system. 
It provides high-level APIs in *Java*, *Scala*, *Python* and *R*, and an optimized engine that supports *general execution graphs*. 
It also supports a rich set of higher-level tools including :

- ***Spark SQL*** for SQL and structured data processing, extends to DataFrames and DataSets
- ***MLlib*** for machine learning
- ***GraphX*** for graph processing
- ***Spark Streaming*** for stream data processing

# spark起源

 - ***2002***  MapReduce @ Google 
 - ***2004***  MapReduce paper 
 -  ***2006***  Hadoop @ Yahoo! 
 -  ***2008***  Hadoop Summit 
 -  ***2009***  started
 -  ***2010***  Spark paper &open sourced
 -  ***2014***  Apache Spark top-level

Spark RDD (Resilient Distributed Datasets) paper/论文：
http://spark.apachecn.org/paper/zh/spark-rdd.html（中文版）
https://www.usenix.org/system/files/conference/nsdi12/nsdi12-final138.pdf（English Version）
 
诞生于2009年，并于次年开源，其目标在于：

 1. 推广MapReduce以支持同一引擎中的新应用程序
 2. 超越hadoop迭代式计算速度

spark能与hadoop完美兼容，能以hadoop集成，并且能通过Mesos&standalone或者在云上部署集群，亦可直接访问HDFS、Cassndra、HBase、S3等数据源；与hadoop不同的是，采用了内存+硬盘取代hadoop只能在硬盘上作为数据存储的方式；设计了新的编程模型RDD,能让数据处理优雅的转换、操作、分布式job、staged、tasks。
 
# spark核心对象
- Master，
- work 
- Executor
- Driver 
- CoarseGrainedExecutorbackend
![master worker executor关系图][1]

从部署图中可以看到

整个集群分为 Master 节点和 Worker 节点，相当于 Hadoop 的 Master 和 Slave 节点。

Master 节点上常驻 Master 守护进程，负责管理全部的 Worker 节点。

Worker 节点上常驻 Worker 守护进程，负责与 Master 节点通信并管理 executors。

Driver 官方解释是 “The process running the main() function of the application and creating the SparkContext”。Application 就是用户自己写的 Spark 程序（driver program），比如 WordCount.scala。如果 driver program 在 Master 上运行，比如在 Master 上运行
```java
  ./bin/run-example SparkPi 10
```
那么 SparkPi 就是 Master 上的 Driver。如果是 YARN 集群，那么 Driver 可能被调度到 Worker 节点上运行（比如上图中的 Worker Node 2）。另外，如果直接在自己的 PC 上运行 driver program，比如在 Eclipse 中运行 driver program，使用
```java
  val sc = new SparkContext("spark://master:7077", "AppName")
```
去连接 master 的话，driver 就在自己的 PC 上，但是不推荐这样的方式，因为 PC 和 Workers 可能不在一个局域网，driver 和 executor 之间的通信会很慢。

每个 Worker 上存在一个或者多个 ExecutorBackend 进程。每个进程包含一个 Executor 对象，该对象持有一个线程池，每个线程可以执行一个 task。

每个 application 包含一个 driver 和多个 executors，每个 executor 里面运行的 tasks 都属于同一个 application。

在 Standalone 版本中，ExecutorBackend 被实例化成 **CoarseGrainedExecutorBackend** 进程。

在我部署的集群中每个 Worker 只运行了一个 CoarseGrainedExecutorBackend 进程，没有发现如何配置多个 CoarseGrainedExecutorBackend 进程。（应该是运行多个 applications 的时候会产生多个进程，这个我还没有实验，）

Worker 通过持有 ExecutorRunner 对象来控制 CoarseGrainedExecutorBackend 的启停。

 在Worker Actor中，每次LaunchExecutor会创建一个CoarseGrainedExecutorBackend进程，Executor和CoarseGrainedExecutorBackend是1对1的关系。也就是说集群里启动多少Executor实例就有多少CoarseGrainedExecutorBackend进程。
 
 
## Application
用户在 spark 上构建的程序，包含了 driver 程序以及在集群上运行的程序代码，物理机器上涉及了 driver，master，worker 三个节点.

## Driver Program
创建 sc ，定义 udf 函数，定义一个 spark 应用程序所需要的三大步骤的逻辑：加载数据集，处理数据，结果展示。

## Cluster Manager
集群的资源管理器，在集群上获取资源的外部服务。 拿 Yarn 举例，客户端程序会向 Yarn 申请计算我这个任务需要多少的 memory，多少 CPU，etc。 然后 Cluster Manager 会通过调度告诉客户端可以使用，然后客户端就可以把程序送到每个 Worker Node 上面去执行了。

## Worker Node
集群中任何一个可以运行spark应用代码的节点。Worker Node就是物理节点，可以在上面启动Executor进程。

## Executor
在每个 Worker Node 上为某应用启动的一个进程，该进程负责运行任务，并且负责将数据存在内存或者磁盘上，每个任务都有各自独立的 Executor。 Executor 是一个执行 Task 的容器。它的主要职责是：

初始化程序要执行的上下文 SparkEnv，解决应用程序需要运行时的 jar 包的依赖，加载类。
同时还有一个 ExecutorBackend 向 cluster manager 汇报当前的任务状态，这一方面有点类似 hadoop的 tasktracker 和 task。
总结：Executor 是一个应用程序运行的监控和执行容器。

## Jobs
包含很多 task 的并行计算，可以认为是 Spark RDD 里面的 action，每个 action 的触发会生成一个job。 用户提交的 Job 会提交给 DAGScheduler，Job 会被分解成 Stage，Stage 会被细化成 Task，Task 简单的说就是在一个数据 partition 上的单个数据处理流程。

**Job、Stage、Task的关系**

job: A job is triggered by an action, like count() or saveAsTextFile(). Click on a job to see information about the stages of tasks inside it. 理解了吗，所谓一个 job，就是由一个 rdd 的 action 触发的动作，可以简单的理解为，**当你需要执行一个 rdd 的 action 的时候，会生成一个 job。**
stage : stage 是一个 job 的组成单位，就是说，一个 job 会被切分成 1 个或 1 个以上的 stage，然后各个 stage 会按照执行顺序依次执行。至于 job 根据什么标准来切分 stage，可以回顾第二篇博文：『 Spark 』2. spark 基本概念解析 

task : A unit of work within a stage, corresponding to one RDD partition。即 stage 下的一个任务执行单元，一般来说，一个 rdd 有多少个 partition，就会有多少个 task，因为每一个 task 只是处理一个 partition 上的数据。

## Stage
一个 Job 会被拆分为多组 Task，每组任务被称为一个 Stage 就像 Map Stage， Reduce Stage。

Stage 的划分在 RDD 的论文中有详细的介绍，简单的说是以 shuffle 和 result 这两种类型来划分。 在 Spark 中有两类 task:

shuffleMapTask

输出是shuffle所需数据, stage的划分也以此为依据，shuffle之前的所有变换是一个stage，shuffle之后的操作是另一个stage。

resultTask

输出是result，比如 rdd.parallize(1 to 10).foreach(println) 这个操作没有shuffle，直接就输出了，那么只有它的task是resultTask，stage也只有一个；如果是rdd.map(x => (x, 1)).reduceByKey(_ + _).foreach(println), 这个job因为有reduce，所以有一个shuffle过程，那么reduceByKey之前的是一个stage，执行shuffleMapTask，输出shuffle所需的数据，reduceByKey到最后是一个stage，直接就输出结果了。如果job中有多次shuffle，那么每个shuffle之前都是一个stage。

## Task
被送到 executor 上的工作单元。

## Partition
Partition 类似 hadoop 的 Split，计算是以 partition 为单位进行的，当然 partition 的划分依据有很多，这是可以自己定义的，像 HDFS 文件，划分的方式就和 MapReduce 一样，以文件的 block 来划分不同的 partition。总而言之，Spark 的 partition 在概念上与 hadoop 中的 split 是相似的，提供了一种划分数据的方式。

## RDD 
每个RDD有5个主要的属性：

- 一组分片（partition），即数据集的基本组成单位
- 一个计算每个分片的函数
- 对parent RDD的依赖，这个依赖描述了RDD之间的lineage
- 对于key-value的RDD，一个Partitioner，这是可选择的
- 一个列表，存储存取每个partition的preferred位置。对于一个HDFS文件来说，存储每个partition所在的块的位置。这也是可选择的
　　
把上面这5个主要的属性总结一下，可以得出RDD的大致概念。首先要知道，RDD大概是这样一种表示数据集的东西，它具有以上列出的一些属性。是spark项目组设计用来表示数据集的一种数据结构。而spark项目组为了让RDD能handle更多的问题，又规定RDD应该是只读的，分区记录的一种数据集合中。可以通过两种方式来创建RDD：一种是基于物理存储中的数据，比如说磁盘上的文件；另一种，也是大多数创建RDD的方式，即通过其他RDD来创建【以后叫做转换】而成。而正因为RDD满足了这么多特性，所以spark把RDD叫做Resilient Distributed Datasets，中文叫做弹性分布式数据集。很多文章都是先讲RDD的定义，概念，再来说RDD的特性。我觉得其实也可以倒过来，通过RDD的特性反过来理解RDD的定义和概念，通过这种由果溯因的方式来理解RDD也未尝不可。反正对我个人而言这种方式是挺好的。

RDD是Spark的核心，也是整个Spark的架构基础，可以总下出几个它的特性来：

- 它是不变的数据结构存储
- 它是支持跨集群的分布式数据结构
- 可以根据数据记录的key对结构进行分区
- 提供了粗粒度的操作，且这些操作都支持分区
- 它将数据存储在内存中，从而提供了低延迟性

## sc.parallelize


----------
parallelize(c, numSlices=None)
Distribute a local Python collection to form an RDD. Using xrange is recommended if the input represents a range for performance.

----------
简单的说，parallelize 就是把 driver 端定义的一个数据集，或者一个获取数据集的生成器，分发到 worker 上的 executor 中，以供后续分析。这种方式在测试代码逻辑时经常用到，但在构建真正的 spark 应用程序时很少会用到，一般都是从 hdfs 或者数据库去读取数据。

## code distribute
提交 spark 应用时，spark 会把应用代码分发到所有的 worker 上面，应用依赖的包需要在所有的worker上都存在，有两种解决 worker 上相关包依赖的问题：

- 选用一些工具统一部署 spark cluster；
- 在提交 spark 应用的时候，指定应用依赖的相关包，把 应用代码，应用依赖包 一起分发到 worker；

## 本地内存与集群内存
所谓本地内存，是指在 driver 端的程序所需要的内存，由 driver 机器提供，一般用来生成测试数据，接受运算结果等； 所谓集群内存，是指提交到集群的作业能够向集群申请的最多内存使用量，一般用来存储关键数据；

## shuffle
shuffle 是两个 stage 之间的数据传输过程。

## What is Lazy Evaluation - 神马叫惰性求值
本来不想叫“惰性求值”的，看到“惰”这个字实在是各种不爽，实际上，我觉得应该叫”后续求值”，”按需计算”，”晚点搞”这类似的，哈哈。这几天一直在想应该怎么简单易懂地来表达Lazy Evaluation这个概念，本来打算引用MongoDB的Cursor来类比一下的，可总觉得还是小题大做了。这个概念就懒得解释了，主要是觉得太简单了，没有必要把事情搞得这么复杂，哈哈。

##  What is Narrow/Wide Dependency - RDD的宽依赖和窄依赖
首先，先从原文看看宽依赖和窄依赖各自的定义。

narrow dependencies: where each partition of the parent RDD is used by at most one partition of the child RDD, wide dependencis, where multiple child partitions may depend on it.

按照这篇RDD论文中文译文的解释，窄依赖是指子RDD的每个分区依赖于常数个父分区（即与数据规模无关）；宽依赖指子RDD的每个分区依赖于所有父RDD分区。暂且不说这样理解是否有偏差，我们先来从两个方面了解下计算一个窄依赖的子RDD和一个宽依赖的RDD时具体都有什么区别，然后再回顾这个定义。
计算方面：

 - 计算窄依赖的子RDD：可以在某一个计算节点上直接通过父RDD的某几块数据（通常是一块）计算得到子RDD某一块的数据；
 - 计算宽依赖的子RDD：子RDD某一块数据的计算必须等到它的父RDD所有数据都计算完成之后才可以进行，而且需要对父RDD的计算结果进行hash并传递到对应的节点之上；

容错恢复方面：

 -  窄依赖：当父RDD的某分片丢失时，只有丢失的那一块数据需要被重新计算；
 -  宽依赖：当父RDD的某分片丢失时，需要把父RDD的所有分区数据重新计算一次，计算量明显比窄依赖情况下大很多；
 
![宽依赖和窄依赖][2]

# 总结
本文内容参考下面链接，作者整理的非常仔细，spark核心概念几乎都有涉及，非常感谢！

# 参考
http://litaotao.github.io/introduction-to-spark?s=inner

  [1]: https://box.kancloud.cn/2015-07-22_55af27d281727.png
  [2]: http://litaotao.github.io/images/spark-rdd-dependency.png
