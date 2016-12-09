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

        - hbase配置外部的zk
            - 配置hbase-env.sh
               export HBASE_MANAGES_ZK=false
               并将他的权限改为a+x
               ```shell
                chmod z+x hbase-env.sh
               ```


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
                            // include start and not include end
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
                                puts.add(put);
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

    # hbase 高级api
        - hbase 中的put, delete, update , checkAndPut , checkAndDelete 是原子操作，也可以在代码中枷锁
        － 将数据的计算移动到服务端，减少数据的返回量
            - filter
                － 使用过滤器或限制列祖的范围，可以控制被返回到了客户端的数据量　
            - 协处理器
                －　数据的处理流程放到客户端执行，返回一个小的处理结果，他类似于小型的MapRedurce框架，框架将工作分发到整个集群，用户在regin服务器上运行自己的代码
                －　引许用户执行region级的操作，跟关系型数据库的出发器类似
                －　用户自己扩展现有的ｒｐｃ协议来映入自己的调用，在Client触发，在服务端执行。
            － 协处理器的运运用场景
                －　权限控制
            － 两个类：　observer and endpoint
                - observer:
                    - 特定事件发生时调用，
                    －　RegionObserver　（与ｒｅｇｉｏｎ紧密相关）
                    －　MasterObserver（ｄｄｌ）
                    －　WALObserver （ｗａｌ）
                -　endpoint
                    - 类似于关系型数据库的存储过程
            - 协处理器的加载
                - 配置文件中加载core-site.xml
        ```java
            package com.hadoop.hbase;
            import java.io.IOException;
            import java.util.ArrayList;
            import java.util.Iterator;
            import java.util.List;
            import org.apache.hadoop.conf.Configuration;
            import org.apache.hadoop.hbase.Cell;
            import org.apache.hadoop.hbase.CellUtil;
            import org.apache.hadoop.hbase.HBaseConfiguration;
            import org.apache.hadoop.hbase.HColumnDescriptor;
            import org.apache.hadoop.hbase.HTableDescriptor;
            import org.apache.hadoop.hbase.TableName;
            import org.apache.hadoop.hbase.client.Admin;
            import org.apache.hadoop.hbase.client.Connection;
            import org.apache.hadoop.hbase.client.ConnectionFactory;
            import org.apache.hadoop.hbase.client.Delete;
            import org.apache.hadoop.hbase.client.Get;
            import org.apache.hadoop.hbase.client.HTable;
            import org.apache.hadoop.hbase.client.Increment;
            import org.apache.hadoop.hbase.client.Put;
            import org.apache.hadoop.hbase.client.Result;
            import org.apache.hadoop.hbase.client.ResultScanner;
            import org.apache.hadoop.hbase.client.Row;
            import org.apache.hadoop.hbase.client.Scan;
            import org.apache.hadoop.hbase.client.Table;
            import org.apache.hadoop.hbase.filter.CompareFilter;
            import org.apache.hadoop.hbase.filter.Filter;
            import org.apache.hadoop.hbase.filter.SubstringComparator;
            import org.apache.hadoop.hbase.filter.ValueFilter;
            //import org.apache.hadoop.hbase.filter.ValueFilter;
            import org.apache.hadoop.hbase.util.Bytes;
            import org.junit.Test;
            public class HbaseDemo02 {
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
                private void result(Result result) {
                    List<Cell> cells = result.listCells();
                    Iterator<Cell> iterator = cells.iterator();
                    while (iterator.hasNext()) {
                        Cell cell = iterator.next();
                        String rowid = Bytes.toString(cell.getRow());
                        String family = Bytes.toString(cell.getFamily());
                        String qualifier = Bytes.toString(cell.getQualifier());
                        // java.util.Date.getTime() or System.currentTimeMillis()
                        // 当前时间和1970-01-01 UTC的时间差，单位毫秒
                        String value = Bytes.toString(cell.getValue());
                        long key = cell.getTimestamp();
                        System.out.println(rowid + ":" + family + ":" + qualifier + ":" + key + ":" + value);
                    }
                }
                @Test
                public void putData() throws Exception {
                    TableName name = TableName.valueOf("ns2:customers");
                    Table table = con.getTable(name);
                    Put put = new Put(Bytes.toBytes("row0"));
                    put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"), System.currentTimeMillis(),
                            Bytes.toBytes("119991"));
                    put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("age"), System.currentTimeMillis(),
                            Bytes.toBytes("@@@88@@"));
                    put.addColumn(Bytes.toBytes("addr"), Bytes.toBytes("address"), System.currentTimeMillis(),
                            Bytes.toBytes("beijingyyrrr"));
                    table.put(put);
                    System.out.println("put data is successful");
                }
                /**
                 * this is hbase version create table: alter 'ns2:customers', NAME =>
                 * 'base', VERSIONS => 3 out put:
                 * row0:addr:address:1479813421026:beijingyyrrr
                 * row0:base:age:1479813421026:@@@88@@ row0:base:age:1479813214796:@@@@@
                 * row0:base:age:1479812850443:**** row0:base:name:1479813421016:119991
                 * row0:base:name:1479813214787:111 row0:base:name:1479812850443:111
                 * 
                 * @throws Exception
                 */
                @Test
                public void getData() throws Exception {
                    TableName name = TableName.valueOf("ns2:customers");
                    Table table = con.getTable(name);
                    Get get = new Get(Bytes.toBytes("row0"));
                    get.setMaxVersions(2);
                    Result result = table.get(get);
                    result(result);
                    System.out.println("put data is successful");
                }
                /**
                 * write buffer
                 */
                @Test
                public void buffer() throws Exception {
                    TableName name = TableName.valueOf("ns2:customers");
                    HTable table = (HTable) con.getTable(name);
                    System.out.println("Auto flush : " + ((HTable) table).isAutoFlush());
                    // set client fresh write false, start client write buffer
                    table.setAutoFlush(false, true);
                    long start = System.currentTimeMillis();
                    List<Put> puts = new ArrayList<Put>();
                    for (int i = 0; i < 10000; i++) {
                        Put put = new Put(Bytes.toBytes("row" + i));
                        put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"), Bytes.toBytes("james" + i));
                        put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("age"), Bytes.toBytes("12" + i));
                        put.addColumn(Bytes.toBytes("addr"), Bytes.toBytes("address"), Bytes.toBytes("beijing" + i));
                        puts.add(put);
                    }
                    table.flushCommits();
                    long end = System.currentTimeMillis();
                    System.out.println(end - start);
                    System.out.println("buffer data is successful");
                }
                // update and insert
                @Test
                public void putDatas() throws Exception {
                    TableName name = TableName.valueOf("ns2:customers");
                    Table table = con.getTable(name);
                    long start = System.currentTimeMillis();
                    List<Put> puts = new ArrayList<Put>();
                    for (int i = 0; i < 10000; i++) {
                        Put put = new Put(Bytes.toBytes("row" + i));
                        put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"), Bytes.toBytes("james" + i));
                        put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("age"), Bytes.toBytes("12" + i));
                        put.addColumn(Bytes.toBytes("addr"), Bytes.toBytes("address"), Bytes.toBytes("beijing" + i));
                        puts.add(put);
                    }
                    table.put(puts);
                    long end = System.currentTimeMillis();
                    System.out.println(end - start);
                    System.out.println("put data is successful");
                }
                // compare-and-set --------
                /**
                 * check and put:  账户结余，状态转化，数据处理
                 *              :  check data and update data
                 *              :  read data and operation data the end write data into database 
                */
                @Test
                public void checkAndPut() throws Exception{
                    TableName name = TableName.valueOf("ns2:customers");
                    Table table =  con.getTable(name);
                    //put data
                    Put put = new Put(Bytes.toBytes("row1"));
                    put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"), Bytes.toBytes("james"));
                    // success return true and fail return false
                    // this is check data
                    boolean  res = table.checkAndPut(Bytes.toBytes("row1"), Bytes.toBytes("base"), Bytes.toBytes("name"), null, put);
                    System.out.println(res);
                    
                    //put data
                    Put put2 = new Put(Bytes.toBytes("row1"));
                    put2.addColumn(Bytes.toBytes("base"), Bytes.toBytes("age"), Bytes.toBytes("45"));
                
                    // success return true and fail return false
                    boolean  res2 = table.checkAndPut(Bytes.toBytes("row1"), Bytes.toBytes("base"), Bytes.toBytes("age"), Bytes.toBytes("12"), put2);
                    System.out.println(res2);
                }
                
                //get Lists
                //compare and delete
                @Test
                public void checkAndDelete() throws Exception{
                    TableName name = TableName.valueOf("ns2:customers");
                    Table table =  con.getTable(name);
                    
                    //delete column 
                    Delete delete = new Delete(Bytes.toBytes("row1"));
                    delete.addColumns(Bytes.toBytes("base"),Bytes.toBytes("name"));
                    
                    //check column data
                    boolean res0 = table.checkAndDelete(Bytes.toBytes("row1"), Bytes.toBytes("base"), Bytes.toBytes("name"),Bytes.toBytes("james") , delete);
                    System.out.println(res0);
                    
                    Delete delete2 = new Delete(Bytes.toBytes("row1"));
                    delete2.addColumns(Bytes.toBytes("base"),Bytes.toBytes("name"));
                    table.delete(delete2);
                    
                    boolean res1 = table.checkAndDelete(Bytes.toBytes("row1"), Bytes.toBytes("base"), Bytes.toBytes("name"),null, delete);
                    System.out.println(res1);
                }
                
                @Test
                public void getList() throws  Exception{
                    TableName name = TableName.valueOf("ns2:customers");
                    Table table =  con.getTable(name);
                    List<Get> gets = new ArrayList<Get>();
                    for(int i = 0 ; i < 2 ; i++){
                        Get get = new Get(Bytes.toBytes("row" + i));
                        gets.add(get);
                    }
                    Result[]  results = table.get(gets);
                    for(Result result: results ){
                        result(result);
                    }
                }
               
               //batch option
               @Test
               public void batch() throws Exception{
                   TableName name = TableName.valueOf("ns2:customers");
                   Table table =  con.getTable(name);
                   List<Row> batch = new ArrayList<Row>();
                   
                   Put put = new Put(Bytes.toBytes("row2"));
                   put.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"), Bytes.toBytes("tomos"));
                   batch.add(put);
                   
                   Delete delete = new Delete(Bytes.toBytes("row0"));
                   delete.addColumn( Bytes.toBytes("base"), Bytes.toBytes("name"));
                   batch.add(delete);
                   
                   Get get = new Get(Bytes.toBytes("row1"));
                   get.addFamily(Bytes.toBytes("base"));
                   batch.add(get);
                   
                   Object[] result = new Object[3];
                   table.batch(batch,result);
                   
                   for(int i = 0 ; i <result.length; i++){
                       System.out.println(result[i]);
                   }
               }
               
               //filter
               @Test
               public void filterScan() throws Exception{
                   TableName name = TableName.valueOf("ns2:customers");
                   Table table =  con.getTable(name);
                   
                   Scan scan  = new Scan();
            //     scan.addColumn(Bytes.toBytes("base"), Bytes.toBytes("name"));
                   scan.addFamily(Bytes.toBytes("base"));
                   
            //     Filter filter = new RowFilter(CompareFilter.CompareOp.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("row1")));
            //     Filter filter = new FamilyFilter(CompareFilter.CompareOp.LESS, new BinaryComparator(Bytes.toBytes("base")));
            //     Filter filter = new QualifierFilter(CompareFilter.CompareOp.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("name")));
                   Filter filter = new ValueFilter(CompareFilter.CompareOp.LESS_OR_EQUAL, new SubstringComparator("m"));
                   
                   scan.setFilter(filter);
                   ResultScanner results = table.getScanner(scan);
                   
                   for(Result result : results){
                       result(result);
                   }
               }
               
               //counter
               @Test
               public void counter() throws Exception{
                   Admin admin = con.getAdmin();
                   TableName name = TableName.valueOf("ns2:counter");
                   HTableDescriptor desc = new HTableDescriptor(name);
                   HColumnDescriptor family = new HColumnDescriptor(Bytes.toBytes("daily"));
                   desc.addFamily(family);
                   admin.createTable(desc);
               }
               
               // hbase counter: one counter
               @Test
               public void incr() throws Exception{
                   TableName name = TableName.valueOf("ns2:counter");   
                   Table table = con.getTable(name);
                   //counter + 1 
                   long result = table.incrementColumnValue(Bytes.toBytes("row1"), Bytes.toBytes("daily"), Bytes.toBytes("hits"), 1);
                   long currentValue = table.incrementColumnValue(Bytes.toBytes("row1"), Bytes.toBytes("daily"), Bytes.toBytes("hits"), 0);
                   // get current counter value
                   System.out.println(result);
                   System.out.println(currentValue);
               }
               
               //hbase counter : more counter
               @Test
               public void moreIncr() throws Exception{
                   TableName name = TableName.valueOf("ns2:counter");   
                   Table table = con.getTable(name);
                   Increment increment = new Increment(Bytes.toBytes("row1"));
                   increment.addColumn(Bytes.toBytes("daily"), Bytes.toBytes("hits"), 1);
                   increment.addColumn(Bytes.toBytes("daily"), Bytes.toBytes("click"), 1);
                   Result result = table.increment(increment);
                   for(Cell  cell : result.rawCells()){
                      System.out.println(Bytes.toLong(CellUtil.cloneValue(cell)));
                   }
               }
               
            }
        ```
    # Hbase mapReduce
