#背景
  -(转载：http://blog.csdn.net/l1028386804/article/details/51935169)
    - Hadoop在设计时考虑到数据的安全与高效，数据文件默认在HDFS上存放三份，存储策略为本地一份，同机架内其它某一节点上一份，不同机架的某一节点上一份。这样如果本地数据损坏，节点可以从同一机架内的相邻节点拿到数据，速度肯定比从跨机架节点上拿数据要快；同时，如果整个机架的网络出现异常，也能保证在其它机架的节点上找到数据。为了降低整体的带宽消耗和读取延时，HDFS会尽量让读取程序读取离它最近的副本。如果在读取程序的同一个机架上有一个副本，那么就读取该副本。如果一个HDFS集群跨越多个数据中心，那么客户端也将首先读本地数据中心的副本。那么Hadoop是如何确定任意两个节点是位于同一机架，还是跨机架的呢？答案就是机架感知。
  
    - 默认情况下，hadoop的机架感知是没有被启用的。所以，在通常情况下，hadoop集群的HDFS在选机器的时候，是随机选择的，也就是说，很有可能在写数据时，hadoop将第一块数据block1写到了rack1上，然后随机的选择下将block2写入到了rack2下，此时两个rack之间产生了数据传输的流量，再接下来，在随机的情况下，又将block3重新又写回了rack1，此时，两个rack之间又产生了一次数据流量。在job处理的数据量非常的大，或者往hadoop推送的数据量非常大的时候，这种情况会造成rack之间的网络流量成倍的上升，成为性能的瓶颈，进而影响作业的性能以至于整个集群的服务

#配置
  - 要将hadoop机架感知的功能启用，配置非常简单，在NameNode所在节点的/home/bigdata/apps/hadoop/etc/hadoop的core-site.xml配置文件中配置一个选项:
  
  ```xml
    <property>  
      <name>topology.script.file.name</name>  
      <value>/home/bigdata/apps/hadoop/etc/hadoop/topology.sh</value>  
    </property>
  ```
  - 这个配置选项的value指定为一个可执行程序，通常为一个脚本，该脚本接受一个参数，输出一个值。接受的参数通常为某台datanode机器的ip地址，而输出的值通常为该ip地址对应的datanode所在的rack，例如”/rack1”。Namenode启动时，会判断该配置选项是否为空，如果非空，则表示已经启用机架感知的配置，此时namenode会根据配置寻找该脚本，并在接收到每一个datanode的heartbeat时，将该datanode的ip地址作为参数传给该脚本运行，并将得到的输出作为该datanode所属的机架ID，保存到内存的一个map中.
  
  - 至于脚本的编写，就需要将真实的网络拓朴和机架信息了解清楚后，通过该脚本能够将机器的ip地址和机器名正确的映射到相应的机架上去.
  
  ```sh
  #!/bin/bash  
  HADOOP_CONF=/home/bigdata/apps/hadoop/etc/hadoop  
  while [ $# -gt 0 ] ; do  
    nodeArg=$1  
    exec<${HADOOP_CONF}/topology.data  
    result=""  
    while read line ; do  
      ar=( $line )  
      if [ "${ar[0]}" = "$nodeArg" ]||[ "${ar[1]}" = "$nodeArg" ]; then  
        result="${ar[2]}"  
      fi  
    done  
    shift  
    if [ -z "$result" ] ; then  
      echo -n "/default-rack"  
    else  
      echo -n "$result"  
    fi  
    done
  ```
 - topology.data,格式为：节点（ip或主机名） /交换机xx/机架xx
 - 192.168.147.91 tbe192168147091 /dc1/rack1
 - 192.168.147.92 tbe192168147092 /dc1/rack1
 - 192.168.147.93 tbe192168147093 /dc1/rack2
 - 192.168.147.94 tbe192168147094 /dc1/rack3
 - 192.168.147.95 tbe192168147095 /dc1/rack3
 - 192.168.147.96 tbe192168147096 /dc1/rack3
 - 需要注意的是，在Namenode上，该文件中的节点必须使用IP，使用主机名无效，而Jobtracker上，该文件中的节点必须使用主机名，使用IP无效,所以，最好ip和主机名都配上。
