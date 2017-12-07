---
layout: post
date: 2017-12-08 11:00
title: Spark world count
description: spark入门，计算文本中词频
categories: [Spark]
tags: [Spark,SBT,Assemblu]
---

# sbt项目
## idea安装scala插件
参考：http://blog.csdn.net/a2011480169/article/details/52712421
# worldCount代码编写
## Scala代码版本
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
## Java代码版本
```java
        /**
         *
         * new FlatMapFunction<String, String>两个string分别代表输入和输出类型
         * Override的call方法需要自己实现一个转换的方法，并返回一个Iterable的结构
         *
         * flatmap属于一类非常常用的spark函数，简单的说作用就是将一条rdd数据使用你定义的函数给分解成多条rdd数据
         * 例如，当前状态下，lines这个rdd类型的变量中，每一条数据都是一行String，我们现在想把他拆分成1个个的词的话，
         * 可以这样写 ：
         */
        //flatMap与map的区别是，对每个输入，flatMap会生成一个或多个的输出，而map只是生成单一的输出
        //用空格分割各个单词,输入一行,输出多个对象,所以用flatMap
        JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String s) {
                return Arrays.asList(SPACE.split(s)).iterator();
            }
        });
        /**
         * map 键值对 ，类似于MR的map方法
         * pairFunction<T,K,V>: T:输入类型；K,V：输出键值对
         * 表示输入类型为T,生成的key-value对中的key类型为k,value类型为v,对本例,T=String, K=String, V=Integer(计数)
         * 需要重写call方法实现转换
         */
        JavaPairRDD<String, Integer> ones = words.mapToPair(new PairFunction<String, String, Integer>() {
            //scala.Tuple2<K,V> call(T t)
            //Tuple2为scala中的一个对象,call方法的输入参数为T,即输入一个单词s,新的Tuple2对象的key为这个单词,计数为1
            @Override
            public Tuple2<String, Integer> call(String s) {
                return new Tuple2<String, Integer>(s, 1);
            }
        });
        //A two-argument function that takes arguments
        // of type T1 and T2 and returns an R.
        /**
         * 调用reduceByKey方法,按key值进行reduce
         *  reduceByKey方法，类似于MR的reduce
         *  要求被操作的数据（即下面实例中的ones）是KV键值对形式，该方法会按照key相同的进行聚合，在两两运算
         *  若ones有<"one", 1>, <"one", 1>,会根据"one"将相同的pair单词个数进行统计,输入为Integer,输出也为Integer
         *输出<"one", 2>
         */
        JavaPairRDD<String, Integer> counts = ones.reduceByKey(new Function2<Integer, Integer, Integer>() {
            //reduce阶段，key相同的value怎么处理的问题
            @Override
            public Integer call(Integer i1, Integer i2) {
                return i1 + i2;
            }
        });
        //备注：spark也有reduce方法，输入数据是RDD类型就可以，不需要键值对，
        // reduce方法会对输入进来的所有数据进行两两运算

        /**
         * collect方法用于将spark的RDD类型转化为我们熟知的java常见类型
         */
        List<Tuple2<String, Integer>> output = counts.collect();
        for (Tuple2<?,?> tuple : output) {
            System.out.println(tuple._1() + ": " + tuple._2());
        }
        ctx.stop();
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
