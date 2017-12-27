---
layout: post
date: 2017-12-25 12:00
title: Spark performance optimization[repost]
description: Spark性能调优。从四个方面着手：开发调优、资源调优、数据倾斜调优、shuffle调优.。开发调优和资源调优是所有Spark作业都需要注意和遵循的一些基本原则，是高性能Spark作业的基础；数据倾斜调优，主要讲解了一套完整的用来解决Spark作业数据倾斜的解决方案；shuffle调优，面向的是对Spark的原理有较深层次掌握和研究的同学，主要讲解了如何对Spark作业的shuffle运行过程以及细节进行调优。
categories: [Spark]
tags: [Spark]
---

调优文章：
----------

- [Spark开发调优][1]
- [Spark资源调优][2]
- [Spark数据倾斜调优][3]
- [Sparkshuffle调优][4]

下面几篇是spark网络上自己学习过程中遇到的总结比较到位，从原理、源码分析以及案例讲解，理解透彻，spark基础和进阶应算是合格吧。
----------
- [spark 原理系列文章1][5]
- [spark 原理系列文章2][6]
- [spark 原理系列文章3][7]
- [spark 原理系列文章3][8]
- [spark 案例等系列文章][9]
- [spark 源码解读][10]

  [1]: https://www.iteblog.com/archives/1657.html
  [2]: https://www.iteblog.com/archives/1659
  [3]: https://www.iteblog.com/archives/1671
  [4]: https://www.iteblog.com/archives/1672
  [5]: http://litaotao.github.io/introduction-to-spark?s=inner
  [6]: https://spark-internals.books.yourtion.com/
  [7]: https://github.com/ColZer/DigAndBuried
  [8]: https://github.com/JerryLead/SparkInternals/tree/master/markdown
  [9]: https://www.iteblog.com/archives/category/spark/
  [10]: https://ihainan.gitbooks.io/spark-source-code/content/section1/rddDependencies.html
