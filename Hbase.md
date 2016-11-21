Hbase:

    - Summary (${**} update your self's setting)
        - set up hbase
           - 解压
           - configuration env
                    - ${hbase_home}
                    - ${hbase_path}
           - update hbase-env.sh
              ```shell
                    export JAVA_HOME=${JAVA_HOME}
              ```
           - standalone_conf
                - make dir
                 ```shell
                     mkdir /opt/hbasedata
                 ```
                 - update hbase-site.xml
                    ```xml
                       <configuration>
                            <property>
                                <name>hbase.rootdir</name>
                                <value>file:///opt/hbase-1.2.x/hbase</value>
                            </property>
                        </configuration>
                    ```
             - pseudo_conf
                ```xml
                    <configuration>
                        <property>
                            <name>hbase.cluster.distributed</name>
                            <value>true</value>
                        </property>
                        <property>
                            <name>hbase.rootdir</name>
                            <value>hdfs://localhost:${port}/hbase</value>
                        </property>
                    </configuration>
                ```
                - cluster_conf
                #### some time pid :export HBASE_PID_DIR=/home/james/pid
                ```shell
                    <property>
                        <name>hbase.rootdir</name>
                        <value>hdfs://192.168.222.140:9000/hbase</value>
                    </property>
                    <property>
                        <name>hbase.cluster.distributed</name>
                        <value>true</value>
                    </property>
                    <property>
                        <name>hbase.master</name>
                        <value>hdfs://gcs-cloud-140:60000</value>
                    </property>
                    <property>
                        <name>hbase.zookeeper.property.clientPort</name>
                        <value>2222</value>
                    </property>
                    <property>
                        <name>hbase.zookeeper.quorum</name>
                        <value>gcs-cloud-114,gcs-cloud-115,gcs-cloud-117</value>
                        <description>节点间不能有空格</description>
                    </property>
                    <property>
                        <name>zookeeper.session.timeout</name>
                        <value>60000</value>
                    </property>
                    <property>
                        <name>hbase.zookeeper.property.dataDir</name>
                        <value>/home/james/zookeeper</value>
                    </property>
                    <property>
                        <name>hbase.master.maxclockskew</name>
                        <value>180000</value>
                        <description>Time difference of regionserver from master,主从机的同步时间差毫秒为单位</description>
                    </property>
                - start up
                   ```shell
                        - start-all.sh 启动hadoop(2.7.3)
                        - start-hbase.sh 启动hbase(1.2.4)
                   ```
                - test
                    - 浏览器 输入 ：http://localhost:16010/master-status
                    - 观察到hbase相关信息

        - Designing tables
            - The number of column families and which data goes to which column family
            - The maximum number of columns in each column family
            - The type of data to be stored in the column
            - The number of historical values that need to be maintained for each column
            - The structure of a rowkey

        - Practices are followed to ensure optimal table design
            - ????

        - Shell options
            ```shell
                create 'test', 'cf'
                list 'test'
                put 'test', 'row1', 'cf:a', 'value1'
                put 'test', 'row2', 'cf:b', 'value2'
                put 'test', 'row3', 'cf:c', 'value3'
                put 'test', 'row4', 'cf:d', 'value4'
                scan 'test'
                get 'test', 'row1'
                disable 'test'
                enable 'test'
            ```
         - hbase java api option
             - create name space
             - droptable
             - tablelist
             - createtable
             - insert date
             - putDate
             - daterow
             - getone
             - findAll
             ```java
                    package com.hbase.demo;
                    import java.io.IOException;
                    import java.util.ArrayList;
                    import java.util.Iterator;
                    import java.util.List;
                    import org.apache.hadoop.conf.Configuration;
                    import org.apache.hadoop.hbase.Cell;
                    import org.apache.hadoop.hbase.HBaseConfiguration;
                    import org.apache.hadoop.hbase.HColumnDescriptor;
                    import org.apache.hadoop.hbase.HRegionInfo;
                    import org.apache.hadoop.hbase.HTableDescriptor;
                    import org.apache.hadoop.hbase.NamespaceDescriptor;
                    import org.apache.hadoop.hbase.TableName;
                    import org.apache.hadoop.hbase.client.Admin;
                    import org.apache.hadoop.hbase.client.Connection;
                    import org.apache.hadoop.hbase.client.ConnectionFactory;
                    import org.apache.hadoop.hbase.client.Delete;
                    import org.apache.hadoop.hbase.client.Get;
                    import org.apache.hadoop.hbase.client.Put;
                    import org.apache.hadoop.hbase.client.Result;
                    import org.apache.hadoop.hbase.client.ResultScanner;
                    import org.apache.hadoop.hbase.client.Scan;
                    import org.apache.hadoop.hbase.client.Table;
                    import org.apache.hadoop.hbase.util.Bytes;
                    import org.junit.Test;
                    public class TestHbase {
                        private static Configuration conf;
                        private static Connection con;
                        private static Admin admin;
                        static {
                            conf = HBaseConfiguration.create();
                            try {
                                con = ConnectionFactory.createConnection(conf);
                                admin = con.getAdmin();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                        /**
                        * create name space
                        * @throws Exception
                        */
                        @Test
                        public void createNamespace() throws Exception{
                           NamespaceDescriptor namespace = NamespaceDescriptor.create("ns1").build();
                           admin.createNamespace(namespace);
                           admin.close();
                           System.out.println("create name space is successful!");
                        }
                        @Test
                        public void dropTable() throws Exception{
                            TableName name = TableName.valueOf("ns1:customers");
                            admin.disableTable(name);
                            admin.deleteTable(name);
                            System.out.println("drop table is successful");
                        }
                        @Test
                        public void createTable() throws Exception{
                            TableName name =  TableName.valueOf("ns1:customers");
                            HTableDescriptor desc = new HTableDescriptor(name);
                            HColumnDescriptor family = new HColumnDescriptor(Bytes.toBytes("base"));
                            desc.addFamily(family);
                            family = new HColumnDescriptor(Bytes.toBytes("addr"));
                            desc.addFamily(family);
                            admin.createTable(desc);
                            System.out.println("drop table is successful");
                        }
                        @Test
                        public void findAll() throws Exception{
                            Scan scan = new Scan(Bytes.toBytes("row1"), Bytes.toBytes("row11"));
                            //scan add filter
                            TableName name = TableName.valueOf("ns1:customers");
                            Table table = con.getTable(name);
                            ResultScanner resultScanner = table.getScanner(scan);
                            Iterator<Result> iterator = resultScanner.iterator();
                            while(iterator.hasNext()){
                                this.result(iterator.next());
                            }
                        }
                        //update and insert
                        @Test
                        public void putData() throws Exception{
                            TableName name = TableName.valueOf("ns1:customers");
                            Table table = con.getTable(name);
                            List<Put> puts = new ArrayList<Put>();
                            for(int i = 0; i < 100; i++){
                                Put put = new Put(Bytes.toBytes("row1" + i));
                                put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"  + i), Bytes.toBytes("james"+ i));
                                put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("age"+ i), Bytes.toBytes("12"+ i));
                                put.addColumn(Bytes.toBytes("addr"), Bytes.toBytes("address"+ i), Bytes.toBytes("beijing"+ i));
                            }
                            table.put(puts);
                            System.out.println("put data is successful");
                        }
                        @Test
                        public void detaData() throws Exception{
                            TableName name = TableName.valueOf("ns1:customers");
                            Table table = con.getTable(name);
                            Delete delete = new Delete(Bytes.toBytes("row1"));
                            table.delete(delete);
                            System.out.println("put data is successful");
                        }
                        @Test
                        public void getData() throws Exception{
                            TableName name = TableName.valueOf("ns1:customers");
                            Table table = con.getTable(name);
                            Get get = new Get(Bytes.toBytes("row1"));
                            Result result = table.get(get);
                            result(result);
                            System.out.println("put data is successful");
                        }
                        private void result(Result result){
                           List<Cell> cells = result.listCells();
                           while(cells.iterator().hasNext()){
                                Cell cell = cells.iterator().next();
                                String rowid = Bytes.toString(cell.getRow());
                                String family = Bytes.toString(cell.getFamily());
                                String qualifier = Bytes.toString(cell.getQualifier());
                                String value = Bytes.toString(cell.getValue());
                                long key = cell.getTimestamp();
                                System.out.println(rowid + ":" + family + ":" + qualifier + ":" + key+ ":" + value);
                           }
                        }
                        public void split() throws Exception{
                            TableName name = TableName.valueOf("ns1:customers");
                            List<HRegionInfo> regionInfor = admin.getTableRegions(name);
                            String regionName = null;
                            for(HRegionInfo infor : regionInfor){
                               long regionId = infor.getRegionId();
                               regionName = Bytes.toString(infor.getRegionName());
                            }
                            //split region
                            admin.splitRegion(Bytes.toBytes(regionName));
                            byte[] a = Bytes.toBytes("dd");
                            byte[] b = Bytes.toBytes("dd");
                            //merge regions
                            admin.mergeRegions(a, b, true);
                        }
                    }
                ```
