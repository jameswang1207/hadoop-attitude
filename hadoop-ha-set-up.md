#　Hadoop ha搭建

### 配置文件

core-site.xml 配置

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
      <!--单个namenode配置-->
      <!-- <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
      </property> -->
      <!--ha: namenode配置-->
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
      </property>
      <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
      </property>
      <property>
        <name>ha.zookeeper.quorum</name>
        <value>gcs-cloud-114:2181,gcs-cloud-115:2181,gcs-cloud-118:2181</value>
      </property>
    </configuration>
```　

hdfs-site.xml　配置 (文件中的不能出现中文配置)

```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  <!--
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License. See accompanying LICENSE file.
  -->

  <!-- Put site-specific property overrides in this file. -->
  <configuration>
    <property>
      <name>dfs.replication</name>
      <value>2</value>
    </property>

    <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>gcs-cloud-118:50090</value>
    </property>

    <property>
      <name>dfs.nameservices</name>
      <value>mycluster</value>
      <description>the logical name for this new nameservice</description>
    </property>
    <property>
      <name>dfs.ha.namenodes.mycluster</name>
      <value>nn1,nn2</value>
      <description>unique identifiers for each NameNode in the nameservice</description>
    </property>
    <property>
      <name>dfs.namenode.rpc-address.mycluster.nn1</name>
      <value>gcs-cloud-117:8020</value>
    </property>
    <property>
      <name>dfs.namenode.rpc-address.mycluster.nn2</name>
      <value>gcs-cloud-140:8020</value>
    </property>
    <property>
      <name>dfs.namenode.http-address.mycluster.nn1</name>
      <value>gcs-cloud-117:50070</value>
    </property>
    <property>
      <name>dfs.namenode.http-address.mycluster.nn2</name>
      <value>gcs-cloud-140:50070</value>
    </property>
    <property>
      <name>dfs.namenode.shared.edits.dir</name>
      <value>qjournal://gcs-cloud-117:8485;gcs-cloud-140:8485;gcs-cloud-118:8485/mycluster</value>
    </property>
    <property>
      <name>dfs.client.failover.proxy.provider.mycluster</name>
      <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <property>
      <name>dfs.journalnode.edits.dir</name>
      <value>/home/james/journal/node/local/data</value>
    </property>
    <property>
      <name>dfs.ha.automatic-failover.enabled</name>
      <value>true</value>
    </property>

    <property>
      <name>dfs.ha.fencing.methods</name>
      <value>sshfence</value>
    </property>
    <property>
      <name>dfs.ha.fencing.ssh.private-key-files</name>
      <value>/home/james/.ssh/id_rsa</value>
    </property>

    <property>
      <name>dfs.namenode.name.dir</name>
      <value>/home/james/dfs/name</value>
    </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>/home/james/dfs/data</value>
    </property>
    <property>
      <name>dfs.namenode.checkpoint.dir</name>
      <value>/home/james/dfs/namesecondary</value>
    </property>

    <property>
      <name>dfs.image.transfer.bandwidthPerSec</name>
      <value>1048576</value>
      <description>
            Maximum bandwidth used for image transfer in bytes per second.
            This can help keep normal namenode operations responsive during
            checkpointing. The maximum bandwidth and timeout in
            dfs.image.transfer.timeout should be set such that normal image
            transfers can complete successfully.
            A default value of 0 indicates that throttling is disabled.
      </description>
  　　</property>
  </configuration>
```

mapred-site.xml　配置

```xml
    <?xml version="1.0"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
        </property>
    </configuration>
```

yarn-site.xml　配置

```xml
  <?xml version="1.0"?>
  <!--
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License. See accompanying LICENSE file.
  -->
  <configuration>
  <!--     <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>gcs-cloud-140</value>
      </property> -->
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>

      <!-- resourcemanager's ha-->
      <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
      </property>
      <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>cluster1</value>
      </property>
      <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
      </property>
      <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>gcs-cloud-117</value>
      </property>
      <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>gcs-cloud-140</value>
      </property>
      <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>mgcs-cloud-117:8088</value>
      </property>
      <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>gcs-cloud-140:8088</value>
      </property>
      <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>gcs-cloud-114:2181,gcs-cloud-115:2181,gcs-cloud-118:2181</value>
      </property>
  </configuration>
```

slaves　配置

```shell
    gcs-cloud-114
    gcs-cloud-115
    gcs-cloud-118
```
启动zookeeper集群（分别在gcs-cloud-114, gcs-cloud-114, gcs-cloud-114上启动zookeeper）并查看zk状态

```shell
 zkServer.sh start
 zkServer.sh status
```
启动journalnode（分别在gcs-cloud-117,gcs-cloud-118,gcs-cloud-140上启动journalnode）注意只有第一次需要这么启动，之后启动hdfs会包含journalnode

格式化HDFS(在gcs-cloud-117上执行)

```shell
# 格式化之后需要把gcs-cloud-140 /home/james/dfs/name目录拷给gcs-cloud-117（不然gcs-cloud-117的namenode起不来）
hdfs namenode -format
scp -r james@gcs-cloud-140:/home/james/dfs/ /home/james/
```
格式化ZKFC

```shell
hdfs zkfc -formatZK
```

启动HDFS(在gcs-cloud-140上执行)
```shell
start-dfs.sh
```

启动YARN(在gcs-cloud-140上执行)
```shell
start-yarn.sh
```
备注
gcs-cloud-117的resourcemanager需要手动单独启动：

```shell
  yarn-daemon.sh start resourcemanager
```
namenode,datanode也可以单独启动：

```shell
  hadoop-daemon.sh start namenode
  hadoop-daemon.sh start datanode
```

NN 由standby转化成active

```shell
  hdfs haadmin -transitionToActive nn1 --forcemanual
```
