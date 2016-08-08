#hadoop 学习笔记

## 初始hadoop

###什么时候使用hadoop
http://just2do.iteye.com/blog/2185254

###硬盘存储容量的不断扩大,访问速度却没有与时俱进.读取大量数据需要较多时间.
>多个磁盘存储同一数据源,并发从多个硬盘上读取数据,缩短读取时间.

###hadoop生态(几个重要工具)
>Ambari: 作为Hadoop生态系统的一部分，这个Apache项目提供了基于Web的直观界面，可用于配置、管理和监控Hadoop集群。有些开发人员想把Ambari的功能整合到自己的应用程序当中，Ambari也为他们提供了充分利用REST（代表性状态传输协议）的API。
>Avro:这个Apache项目提供了数据序列化系统，拥有丰富的数据结构和紧凑格式。模式用JSON来定义，它很容易与动态语言整合起来。
>HDFS:HDFS是面向Hadoop的文件系统，不过它也可以用作一种独立的分布式文件系统。它基于Java，具有容错性、高度扩展性和高度配置性。
>Hive:Apache Hive是面向Hadoop生态系统的数据仓库。它让用户可以使用HiveQL查询和管理大数据，这是一种类似SQL的语言。
>MapReduce:作为Hadoop一个不可或缺的部分，MapReduce这种编程模型为处理大型分布式数据集提供了一种方法。它最初是由谷歌开发的，但现:在也被本文介绍的另外几个大数据工具所使用，包括CouchDB、MongoDB和Riak。
>Spark:作为MapReduce之外的一种选择，Spark是一种数据处理引擎。它声称，用在内存中时，其速度比MapReduce最多快100倍；用在磁盘上时，其速度比MapReduce最多快10倍。它可以与Hadoop和Apache Mesos一起使用，也可以独立使用。
