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

### 二、 HDFS概念
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
- 数据冗余,硬件容错
- 流式数据访问
- 存储大文件

#### 实用性和局限性
-  适合数据的批量读写,吞吐量高
-  不适合交互式应用,底延时
-  一次写入,多次读取,顺序读写
-  不支持多个用户同时写同一个文件

### 三、 HDFS的命令行接口
- http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html
   
### 四、 通过代码访问文件系统(hadoop使用java写的)(java接口)
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
   
   <!-- 通过FileSystem API读取数据 -->
   public class FileSystemCat {
      public static void main(String[] args) throws Exception {
         String uri = args[0];
         Configuration conf = new Configuration();
         FileSystem fs = FileSystem.get(URI.create(uri), conf);
         InputStream in = null;
         try {
            in = fs.open(new Path(uri));
            <!--
            in.seek(0);
            fs.open(new Path(uri))返回的就是FSDataInputStream对象
            通过FSDataInputStream API读取数据:支持随机访问,
            1.由此可以从流的任意位置读取数据.
            2.从指定的文件偏移量来读取数据
            -->
            IOUtils.copyBytes(in, System.out, 4096, false);
         } finally {
            IOUtils.closeStream(in);
         }
      }
   }
  ```
- 文件目录:(FileSystem)
   - 文件的创建
      - FileSystem.mkdirs();
   - 查看文件系统
      - FileStatus:封装了文件的文件长度|块大小|副本|修改时间|所有者以及权限信息
   - 文件过滤|文件删除....
  

#### 通过http来访问HDFS:(两种方式)
- 直接访问.(DistributedFileSystem api访问)
- 通过代理访问(中间加入代理服务器).

## hadoop 的I/O操作
### 数据完整性
- datanode在收到数据后进行数据的存储及验证校验和,并从管线中发送到各个datanode中.
- 读取数据时会和datanode中存储的检验和进行对比.
- 每个datanode会在后台线程中运行一个DataBlockScanner,定期验证存储在上面的所有数据块.
- 错误后的做法
- 客户端通过LocalFileSystem or  checksunFileSystem (校验和计算).

### 文件压缩和解压:减少存储空间,加快数据在网络间和磁盘上的传输.

   - what
      - 在进行文件压缩时,要了解这些压缩格式是否支持切分.
      - 应该使用那种压缩格式?(gzip)
        - 使用容器文件格式(这些都是同时支持压缩和切分的)
           - RCFile是Hive推出的一种专门面向列的数据格式.它遵循“先按列划分，再垂直划分”的设计理念。当查询过程中，针对它并不关心的列时，它会在IO上跳过这些列
           - Avro
      - 对于大文件,不要使用不支持切分整个文件的压缩格式.

   -  how
      - 对mapReduce输出进行压缩.(设置属性)
      - 对mapReduce的map任务的输出进行压缩.(好处)

### 结构化对象  (反序列化) <----> (序列化)字节流
   - where
      - 分布式数据处理的进程间的通讯和永久存储.
         - RPC(romote procedure call):hadoop中多个节点上进程间的通讯.
            - 序列化发送到远程节点,远程节点通过反序列化转化为原始消息.
   - how
      - hadoop自己的序列化格式Writable:但是不易被java以外的语言进行扩展.
         - Writable.........
   
   - 序列化系统:Avro(一种语言写入另一种语言读取)
      - what
         - hadoop中的Writable的不足,缺乏语言的移值性.它拥有一种被多种语言处理的数据格式.
         - 数据格式用语言无关的模式定义. 
         - 模式通常使用JSON来定义.
         - 支持压缩和切分.
      - where
         - 内存中的序列化和反序列化(Avro提供了api)(文件模式的扩展名为avsc).
            - 存储文件格式为json的模式文件后缀为avsc. 
         
            ```json
               {
                 "type": "record",
                 "name": "StringPair",
                 "doc": "A pair of strings.",
                 "fields": [
                   {"name": "left", "type": "string"},
                   {"name": "right", "type": "string"}
                 ]
               }
            ```
      - how 
         - 写入流中
         - 写入数据文件
         - MapReduce的运用.
            
   -  基于文件的数据结构:一些应用中,我们需要特殊的数据结构来存储自己的数据.
      -  SequenceFile
      -  MapFile
         
## hadoop MapReduce
### 经典的mapReduce  vs  YARN
#### 概念
   - 用于数据处理的编程模型
   - 用于处理大规模的数据集
   
#### MapReduce的工作机制:
   - what:
      - 例子:1000副扑克拍:将牌分成n份,每个人统计每个花色数字出现的次数(map)|(数据交换shulft)|进行归并(相同花色的相同数字放在一起(reduce)),找出结果.
      - 分而治之的方法,将一个大任务分成多个小任务(map),并行执行后,合并结果(reduce) .
      
#### 经典模型(mapperReduce)

  - 经典的mapperReduce包含的实体 
      - 客户端(提交mapperReduce作业)
      - jobtracker,协调作业的运行(java 运用程序) 类名JobTracker
      - tasktracker,运行作业划分后的任务(java 运用程序) 类名TaskTracker
      - 分布式文件系统
   
  - mapReduce 运行的流程
      - mapReduce工作原理
         - 图
         
         - 作业放在队列中
            - JobTracker:
               - 作业调度
               - 分配任务
               - 监控任务的执行进度
               - 监控taskTrack的状态
               - 重启失败或过慢的任务
               - 进程任务登记
               
            - TaskTracker:执行任务,向JobTracker发送自己当前的状态
         - 将TaskTrack与node放在一个节点上(task可以很块获得数数)
      
      - mapReduce的作业的执行过程:
         - 作业提交
         - 作业初始化
         - 任务分配
         - 任务执行
         - 进度和状态跟新
         - 作业完成
         
#### YARN (mapperReduce) -> (yet Another Resource Negotiator):另一种资源管理协调者
   - what 
      - YARN : 新的 Hadoop 资源管理器，它是一个通用资源管理系统.
         - http://baike.baidu.com/link?url=OLm0t88P-HB3mcIvEyxM58lrO6GiyNP-VybSMUg-rEz4Kk5UUpDuE1P70q8HCDjxiOTc0PhV7VNf80xqzLXJy_
         - YARN的基本思想是将JobTracker的两个主要功能（资源管理和作业调度/监控）分离.(YARN的主要架构)
            - ResourceManager(RM)
               一个全局的资源管理器，负责整个系统的资源管理和分配.
                  - 调度器: 纯调度器.
                  - 应用程序管理器:整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等.
            - ApplicationMaster(AM)
               - 用户提交的每个应用程序均包含一个AM
                  - 与RM调度器协商以获取资源
                  - 将得到的任务进一步分配给内部的任务
                  - 与NM通信以启动/停止任务
                  - 监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务
                  - RM只负责监控AM，在AM运行失败时候启动它，RM并不负责AM内部任务的容错，这由AM来完成
            - NodeManager(NM)
               - NM是每个节点上的资源和任务管理器，一方面，它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态；另一方面，它接收并处理来自AM的Container启动/停止等各种请求。
            - Container
               - Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是用Container表示。YARN会为每个任务分配一个Container，且该任务只能使用该Container中描述的资源
   - how
      - how to use it : mapred-site.xml
      ```xml
         <configuration>
            <property>
               <name>mapreduce.framework.name</name>
               <value>yarn</value>
            </property>
         </configuration>
      ```
      
   - mapReduce2 运行的流程
       
     
### MapReduce 应用开发:
   #### 配置 合并
      - 通过Configuration来对配置的资源进行整合
       
      ```java
         Configuration conf = new Configuration();
         conf.addResource("conf1.xml");
         conf.addResource("conf2.xml");
         
         <!--conf.getInt(name);-->
      ```
      - 辅助类运行hadoop程序:
         - Tool
         - ToolRunner
      
      - 用MRUnit来进行测试
      
      
   




























