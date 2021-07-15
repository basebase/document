#### flink入门-table程序结构

##### 概述

在使用Flink Table & SQL程序时, 我们也会有一个主体框架结构, 大致可以分为下面几步
1. 创建一个Table执行环境即TableEnvironment实例
2. 从哪里读取数据, 需要创建一张数据源表, 而不是继续从DataStream中读取数据
3. 数据结果输出到哪里去, 创建一张输出表
4. 对数据源表进行查询、选取、过滤等
5. 将结果数据输出到对应输出表


```java

// 1. 创建TableEnvironment实例
StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// 2. 创建一张数据源表
tableEnv.executeSql("CREATE TEMPORARY TABLE soruce_table ... WITH ( 'connector' = ... )");

// 3. 创建一张输出表
tableEnv.executeSql("CREATE TEMPORARY TABLE output_table ... WITH ( 'connector' = ... )");

// 4. 对表进行查询过滤等
// 4.1 使用Table API查询, 返回一个Table对象
Table table1 = tableEnv.from("soruce_table").select("...").where("...");
// 4.2 使用SQL查询, 返回一个Table对象
Table table2 = tableEnv.sqlQuery("select ... from source_table where ...");

// 5. 查询结果输出目标数据源
// 使用Table API将结果写入到目标数据源中
table1.executeInsert("output_table");
```