## HIVE
    - Data file content
        Michael|Montreal,Toronto|Male,30|DB:80|Product:Developer^DLead
        Will|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
        Shelley|New York|Female,27|Python:80|Test:Lead,COE:Architect
        Lucy|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
    - Create table
        CREATE  TABLE   employee
        (
          name string,
          work_place ARRAY<string>,
          sex_age STRUCT<sex:string,age:int>,
          skills_score MAP<string,int>,
          depart_title MAP<string,ARRAY<string>>
        )
        ROW FORMAT DELIMITED
        FIELDS TERMINATED BY '|'
        COLLECTION ITEMS TERMINATED BY ','
        MAP KEYS TERMINATED BY ':';
    - Load data
        - load data local input 'file path' overwrite into table table_name
        - ef: load data local inpath '/home/james/Desktop/hadoop-relevant/employee' overwrite into table employee;

### HIVE PARTITION
    - Create partition table
        CREATE TABLE employee_partitioned
          (
              name string,
              work_place ARRAY<string>,
              sex_age STRUCT<sex:string,age:int>,
              skills_score MAP<string,int>,
              depart_title MAP<STRING,ARRAY<STRING>>
          )
          PARTITIONED BY (Year INT, Month INT)
          ROW FORMAT DELIMITED
          FIELDS TERMINATED BY '|'
          COLLECTION ITEMS TERMINATED BY ','
          MAP KEYS TERMINATED BY ':';

    - Add multiple partitions
            ALTER TABLE employee_partitioned ADD
               PARTITION   (year=2014, month=11)
               PARTITION (year=2014, month=12);
    - Show partition
            ef : SHOW PARTITIONS employee_partitioned;
    - Drop partition
            ef : alter table employee_partitioned drop if exists partition(year=2014,month=11)
    - Load data to the partition:
        - load data local inpath file_path overwrite into table  table_name partition (year=2214,month = 12)
        - ef : load data local inpath "/home/james/Destop/employee" overwrite into table  table_name partition (year=2214,month = 12)

### HIVE BUCKETS
    -Create  the buckets table
          CREATE TABLE employee_id_buckets
          (
              name string,
              employee_id int,
              work_place ARRAY<string>,
              sex_age STRUCT<sex:string,age:int>,
              skills_score MAP<string,int>,
              depart_title MAP<string,ARRAY<string >>
          )
          CLUSTERED BY (employee_id) INTO 2 BUCKETS
          ROW FORMAT DELIMITED
          FIELDS  TERMINATED BY '|'
          COLLECTION ITEMS TERMINATED BY ','
          MAP KEYS TERMINATED BY ':';

### hive 子查询
    - WITH t1 AS ( SELECT * FROM employee WHERE sex_age.sex = 'Male') SELECT name, sex_age.sex AS sex FROM t1;
    - SELECT name, sex_age.sex AS sex FROM (SELECT * FROM employee WHERE sex_age.sex = 'Male')t1;
    - SELECT name, sex_age.sex AS sex FROM employee a WHERE a.name IN (SELECT name FROM employee WHERE sex_age.sex = 'Male');

    ### join 
    - SELECT emp.name, emph.sin_number FROM employee emp JOIN employee_hr emph ON emp.name = emph.name;
    - Self-join
        - ef: SELECT emp.name FROM employee emp JOIN employee emp_b ON emp.name = emp_b.name;

    ### Special JOIN – MAP JOIN :only map without the reduce job
    - ef: SELECT /*+ MAPJOIN(employee) */ emp.name, emph.sin_number FROM employee emp CROSS JOIN employee_hr emph WHERE emp.name <> emph.name;
    - The  MAPJOIN  operation does  not support the following:
        - The use of MAPJOIN after UNION ALL, LATERAL VIEW, GROUP BY / JOIN /SORT BY / CLUSTER  BY / DISTRIBUTE BY
        - The use of MAPJOIN  before UNION , JOIN , and another MAPJOIN

### hive conf mysql
    - important step
     ```shell
        cd ${hive_home}/bin
        ./schematool -dbType mysql -init schema
     ```
    - update hive-site.xml
        ```xml
         <property>
           <name>javax.jdo.option.ConnectionDriverName</name>
           <value>com.mysql.jdbc.Driver</value>
         </property>
         <property>
            <name>javax.jdo.option.ConnectionURL</name>
            <value>jdbc:mysql://127.0.0.1:3306/mybase?characterEncoding=UTF-8</value>
          </property>
          <property>
            <name>javax.jdo.option.ConnectionUserName</name>
            <value>${os username}</value>
          </property>
          <property>
            <name>javax.jdo.option.ConnectionPassword</name>
            <value>${os password}</value>

</property>
        ```

###hive service
    - hive:
      - 基于apache的thrift.base的客户端技术
      - 要求直接链接到hive的drive上，要求安装hive
    - beeline:
      - 基于jdbc的链接，通过jdbc链接到hiveservice2服务器，不需要安装hive类库，可以远程链接


    - beeline option hive:
      - 启动service2
      ```shell
        cd ${hive_home}/bin
        ./hiveserver2
      ```
      - start beeline terminal
        ```shell
          cd ${hive_home}/bin
          ./beeline
        ```
    - into beeline
       ```shell
          !connect jdbc:hive2://${ip}:10000/default
          enter username: ${os username}
          enter password: ${os password}
       ```
    - update hive-site.xml
      ```xml
        <property>
          <name>hive.server2.enable.doAs</name>
          <value>false</value>
        </property>
        <property>
          <name>hive.metastore.sasl.enabled</name>
          <value>false</value>
        </property>
        <property>
          <name>hive.server2.authentication</name>
          <value>NONE</value>
        </property>
      ```
### ide
    - add hive jar and hadoop jar
    ```java
      package com.hadoop.hive;
      import java.sql.Connection;
      import java.sql.DriverManager;
      import java.sql.PreparedStatement;
      public class HiveCreateDb {
        public static void main(String[] args) throws Exception{
             Class<?> clazz = Class.forName("org.apache.hive.jdbc.HiveDriver");
             Connection con = DriverManager.getConnection("jdbc:hive2://192.168.222.140:10000/default", "xxxx", "xxxx"
                + "");
             PreparedStatement pst = con.prepareStatement("insert into infor values (?,?,?)");
             pst.setInt(1, 13);
             pst.setString(2, "james");
             pst.setInt(3, 25);
             pst.execute();
             pst.close();
             System.out.println("over");
           }
      }
    ```
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




