---
layout: post
date: 2017-12-07 11:00
title: Spark world count
description:Spark计算单词个数
categories: [Spark]
tags: [Spark,SBT,Assembly]
---

# sbt项目
## idea安装scala插件
参考：http://blog.csdn.net/a2011480169/article/details/52712421
# worldCount代码编写
## scala代码
```java
Main.scala

object Main {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    val sc = new SparkContext(conf)
    //test.txt在linux上的存放位置
    val line = sc.textFile("/usr/local/spark/spark-2.2.0-bin-hadoop2.7/test.txt") 
    line.flatMap(_.split(" ")).map((_,1)).reduceByKey(_ + _).collect().foreach(println)
    sc.stop()
  }
}
```
## spark&scala兼容性问题
我的spark版本是2.2.0，spark中jars中能查看到其运行使用的scala类库编译器都是2.11.8,如下：
- scala-compiler-2.11.8.jar
- scala-library-2.11.8.jar
- scalap-2.11.8.jar
- scala-reflect-2.11.8.jar

因此在idea中项目中build.sbt指定scalaVersion := "2.11.8"应是最合适的。
我的build.sbt配置如下：
```java
name := "helloworld"

version := "1.0"

scalaVersion := "2.11.8"

assemblyMergeStrategy in assembly := {
  case PathList("org", "apache", xs @ _*)         => MergeStrategy.first
  case PathList("javax", "inject", xs @ _*)         => MergeStrategy.first
  case PathList("org", "aopalliance", xs @ _*)         => MergeStrategy.first
  case PathList(ps@_*) if ps.last endsWith "axiom.xml" => MergeStrategy.filterDistinctLines
  case PathList(ps@_*) if ps.last endsWith "Log$Logger.class" => MergeStrategy.first
  case PathList(ps@_*) if ps.last endsWith "ILoggerFactory.class" => MergeStrategy.first
  case x =>
    val oldStrategy = (assemblyMergeStrategy in assembly).value
    oldStrategy(x)
}

libraryDependencies += "org.apache.spark" % "spark-core_2.11" % "2.2.0"

libraryDependencies += "org.elasticsearch" % "elasticsearch-hadoop" % "6.0.0-rc2"
```
## 项目打包
出了利用IDE工具打包外，另一不错的选择便是通过sbt提供的插件assembly。首先在项目中project文件夹下新建plugins.sbt,附上一下配置：
```java
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.6")

logLevel := sbt.Level.Info //日志级别，可有可无
```
然后在项目根目录运行：
```java
sbt clean assembly
```
helloworld-assembly-1.0.jar有90M大小,OMG...
# spark-submit运行
我的spark安装在：
/usr/local/spark/spark-2.2.0-bin-hadoop2.7/
然后仙剑test.txt文本，内容为：hello world hello spark
在spark根目录运行如下命令：
```java
bin/spark-submit --class Main helloworld-assembly-1.0.jar
```
输出：
```scala
(spark,1)
(hello,2)
(world,1)
```
# worldCount项目源码
https://github.com/yanlzhl/spark-worldCount
