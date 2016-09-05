#背景
  ####(转载：http://blog.csdn.net/l1028386804/article/details/51935169)
  - Hadoop在设计时考虑到数据的安全与高效，数据文件默认在HDFS上存放三份，存储策略为本地一份，同机架内其它某一节点上一份，不同机架的某一节点上一份。这样如果本地数据损坏，节点可以从同一机架内的相邻节点拿到数据，速度肯定比从跨机架节点上拿数据要快；同时，如果整个机架的网络出现异常，也能保证在其它机架的节点上找到数据。为了降低整体的带宽消耗和读取延时，HDFS会尽量让读取程序读取离它最近的副本。如果在读取程序的同一个机架上有一个副本，那么就读取该副本。如果一个HDFS集群跨越多个数据中心，那么客户端也将首先读本地数据中心的副本。那么Hadoop是如何确定任意两个节点是位于同一机架，还是跨机架的呢？答案就是机架感知。
  
  - 默认情况下，hadoop的机架感知是没有被启用的。所以，在通常情况下，hadoop集群的HDFS在选机器的时候，是随机选择的，也就是说，很有可能在写数据时，hadoop将第一块数据block1写到了rack1上，然后随机的选择下将block2写入到了rack2下，此时两个rack之间产生了数据传输的流量，再接下来，在随机的情况下，又将block3重新又写回了rack1，此时，两个rack之间又产生了一次数据流量。在job处理的数据量非常的大，或者往hadoop推送的数据量非常大的时候，这种情况会造成rack之间的网络流量成倍的上升，成为性能的瓶颈，进而影响作业的性能以至于整个集群的服务

#配置
  #### 要将hadoop机架感知的功能启用，配置非常简单，在NameNode所在节点的/home/bigdata/apps/hadoop/etc/hadoop的core-site.xml配置文件中配置一个选项:
  
  ```xml
    <property>  
      <name>topology.script.file.name</name>  
      <value>/home/bigdata/apps/hadoop/etc/hadoop/topology.sh</value>  
    </property>
  ```

