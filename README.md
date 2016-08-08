#hadoop 学习笔记

## 初始hadoop

### 一、什么时候使用hadoop
http://just2do.iteye.com/blog/2185254

### 二、硬盘存储容量的不断扩大,访问速度却没有与时俱进.读取大量数据需要较多时间.
- 多个磁盘存储同一数据源,并发从多个硬盘上读取数据,缩短读取时间.

### 三、hadoop生态(几个重要工具)
- Ambari: 作为Hadoop生态系统的一部分，这个Apache项目提供了基于Web的直观界面，可用于配置、管理和监控Hadoop集群。有些开发人员想把Ambari的功能整合到自己的应用程序当中，Ambari也为他们提供了充分利用REST（代表性状态传输协议）的API。

- Avro:这个Apache项目提供了数据序列化系统，拥有丰富的数据结构和紧凑格式。模式用JSON来定义，它很容易与动态语言整合起来。

- HDFS:HDFS是面向Hadoop的文件系统，不过它也可以用作一种独立的分布式文件系统。它基于Java，具有容错性、高度扩展性和高度配置性。

- Hive:Apache Hive是面向Hadoop生态系统的数据仓库。它让用户可以使用HiveQL查询和管理大数据，这是一种类似SQL的语言。

- MapReduce:作为Hadoop一个不可或缺的部分，MapReduce这种编程模型为处理大型分布式数据集提供了一种方法。它最初是由谷歌开发的，但现:在也被本文介绍的另外几个大数据工具所使用，包括CouchDB、MongoDB和Riak。

- Spark:作为MapReduce之外的一种选择，Spark是一种数据处理引擎。它声称，用在内存中时，其速度比MapReduce最多快100倍；用在磁盘上时，其速度比MapReduce最多快10倍。它可以与Hadoop和Apache Mesos一起使用，也可以独立使用。


## hadoop 分布式文件系统(HDFS)

### 一、 HDFS的设计
#### 当数据集超过一个单独的物理计算机的存储能力时，便有必要将它分不到多个独立的计算机上。管理着跨计算机网络存储的文件系统称为分布式文件系统。Hadoop 的分布式文件系统称为 HDFS，它 是为 以流式数据访问模式存储超大文件而设计的文件系统。
- 超大文件”是指几百 TB 大小甚至 PB 级的数据
- 流式数据访问：HDFS 建立在这样一个思想上一次写入、多次读取的模式是最高效的。一个数据集通常由数据源生成或者复制，接着在此基础上进行各种各样的分析。HDFS 是为了达到高数据吞吐量而优化的，这有可能以延迟为代价。对于低延迟访问，HBase 是更好的选择(如果要对写入的数据进行更改,只能进行删除数据块,重新追加)。
- 商用硬件：即各种零售店都能买到的普通硬件。这种集群的节点故障率蛮高，HDFD需要能应对这种故障。

#### HDFS 还不合适某些领域
- 低延迟数据访问：需要低延迟数据访问在毫秒范围内的应用不合适 HDFS.
- 大量的小文件：HDFS 的 NameNode 存储着文件系统的元数据，因此文件数量的限制也由NameNode 的内存量决定
- 多用户写入、任意修改文件：HDFS中的文件只有一个写入者，而且写操作总是在文件的末尾。它不支持多个写入者，或者在文件的任意位置修改。

### 一、 HDFS概念
#### 数据块
- 块是磁盘读/写数据的最小单位,文件系统块的大小一般为几千个字节,而磁盘块的一般为512字节.
- HDFS默认的块的大小为64mb,HDFS上的文件被划分为快大小的多个分块,作为独立的存储单元.HDFS中小于一块的文件不会占据整块的文件空间.(为什么HDFS中的块如此大?-----)
- 一般情况下HDFS使用128MB的块设置(这样设置的好处?----).

#### 客户端:代表用户通过namenode和datanode交互来访问整个文件系统.

#### namenode:管理节点存放文件元数据

- 文件与数据块的映射表
- 数据块与数据节点间的映射表

#### datanode:HDFS的工作节点,存放数据块

#### 数据的管理策略:
- 图片

#### HDFS中文件的读写操作
- 图片

#### HDFS的特点
- 图片

### 二、 HDFS的命令行接口
   
### 三、 通过代码访问文件系统(hadoop使用java写的)(java接口)
  ```java
     <!--从Hadoop URL读取数据-->
   public class URLCat {
   
     static {
      <!--每个java虚拟机只能调用一次该方法-->
      URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
     }
     
     public static void main(String[] args) throws Exception {
         InputStream in = null;
         try {
            in = new URL(args[0]).openStream();
            IOUtils.copyBytes(in, System.out, 4096, false);
         } finally {
            IOUtils.closeStream(in);
         }
      }
   }
  ```

#### 通过http来访问HDFS:(两种方式)
- 直接访问.
- 通过代理访问.



























