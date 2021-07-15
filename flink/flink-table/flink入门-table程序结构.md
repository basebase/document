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


##### 创建TableEnvironment实例

Flink Old Planner和blink Planner两种计划期, flink1.11及之后的版本默认使用blink, 如果想使用old planner如何创建呢?
针对此问题, 我们下面将熟悉如何创建old和blink两种Planner以及如何创建stream和batch模式

```java
public class FlinkTableEnvironment {
    public static void main(String[] args) {

        /***
         *      flink1.11默认使用的是blink的版本, 但是如果想使用old planner怎么做呢
         */

        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        // 0. 这一步操作, 默认使用blink, 如果想使用old planner如何处理呢
        StreamTableEnvironment.create(env);

        // 1. 在创建TableEnvironment实例, 我们可以添加相关参数用于配置, 创建EnvironmentSettings实例
        EnvironmentSettings oldStreamSettings = EnvironmentSettings.newInstance()
                .useOldPlanner()        // 使用 old planner
                .inStreamingMode()      // stream模式
                .build();

        // 1.1 创建一个基于old planner 的流Table环境
        StreamTableEnvironment oldStreamTableEnv = StreamTableEnvironment.create(env, oldStreamSettings);


        // 2. 创建一个基于old planner的批处理环境
        ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment oldBatchTableEnv = BatchTableEnvironment.create(batchEnv);


        /***
         *
         *      基于blink planner创建流&批的环境
         */


        // 3. 创建基于blink的流环境
        EnvironmentSettings blinkStreamSettings = EnvironmentSettings.newInstance()
                .useBlinkPlanner()      // 使用blink planner
                .inStreamingMode()      // stream模式
                .build();
        StreamTableEnvironment blinkStreamTableEnv = StreamTableEnvironment.create(env, blinkStreamSettings);


        // 4. 创建基于blink的批环境
        EnvironmentSettings blinkBatchSettings = EnvironmentSettings.newInstance()
                .useBlinkPlanner()
                .inBatchMode()
                .build();

        TableEnvironment blinkBatchTableEnv = TableEnvironment.create(blinkBatchSettings);
    }
}
```