#### flink入门-table读取文件数据

##### 实战
前面我们已经清楚的知道如何创建一个Table API & SQL程序的结构, 现在我们需要去读取一个文件中的数据。但是要读取数据我们需要将数据注册成一张表, 并且还要定义其schema以及如何格式化和文件路径等配置信息。

在读取文件之前呢, 我们需要引入一个连接器依赖, 由于是文件连接, 所以需要使用FileSystem SQL Connector

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-filesystem_${scala.binary.version}</artifactId>
    <version>1.11.0</version>
</dependency>
```

有了连接器之后, 但是我们的文件需要按照上面格式进行格式化呢? 这里我采用的是csv格式化的方式, 所以还需要加入csv格式化依赖
```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-csv</artifactId>
    <version>${flink.version}</version>
    <!--<scope>test</scope>-->
</dependency>
```

1. 创建一张表

```java
String sourceTable = "create temporary table source_table (\n" +
                "    id int,\n" +
                "    name string,\n" +
                "    score double\n" +
                ") with (\n" +
                "    'connector'='filesystem',\n" +
                "    'path'= '"+ inPath +"',\n" +
                "    'format'='csv'\n" +
                ")";
tableEnv.executeSql(sourceTable);
```

这里创建一张临时表, 并指定了对应的文件路径以及是有csv的方式进行格式化。


2. 将文件数据查询并打印
```java
String querySQL = "select * from source_table";
Table result = tableEnv.sqlQuery(querySQL);

result.printSchema();
tableEnv.toAppendStream(result, Row.class).print("result");
```

通过SQL语句可以将文件中的数据全部查询出来并将其打印。

****注意: 在旧版本flink中有些是通过connect来创建表的, 在flink1.11之后就被废弃了, 而是通过上面的方式创建表***

```java
// 旧版本创建表
tableEnv.connect()
        .withSchema()
        .withFormat()
        .createTemporaryTable();
```