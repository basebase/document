#### flink入门-table快速入门

##### 概念
Flink Table API和SQL集成在同一套API中, 这套API的核心概念是Table, 用作查询的输入和输出。
这里会大概对Table API和SQL查询程序的通用结构、如何注册Table、如何查询Table以及如何输出Table进行介绍。

##### 如何使用Table&SQL
想要使用Flink Table和SQL需要引入相关依赖, Flink提供了两个Table Planner来实现执行Table API和SQL程序
Blink Planner和Old Planner, Old Planner在1.9之前就已经存在了。Planner的作用主要是把关系型的操作翻译成可执行的、经过优化的Flink任务。

两种Planner所使用的优化规则以及运行时类都不一样。它们在支持的功能上也有些差异。

***注意: flink1.11版本之后默认使用Blink Planner***

我们可以在pom.xml文件中添加相关依赖
```xml
<!-- flink table & sql依赖 old版本 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
    <scope>provided</scope>
</dependency>


<!-- flink table & sql依赖 blink版本 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner-blink_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
    <!--<scope>test</scope>-->
</dependency>
```


##### Table API & SQL程序结构
和我们在开发stream程序一样, 该有的主体结构还是都存在, 但是会多一点内容, 那就是Table如何执行。就和我们执行stream程序需要一个上下文环境, 我们也要有这么个环境可以注册表、执行sql等内容

创建一个TableEnvironment对象实例, 如果我们将两种Planner都添加进依赖, 我们需要明确设置当前程序使用的Planner

```java
EnvironmentSettings settings = EnvironmentSettings
                .newInstance()
                .useBlinkPlanner()
                .inStreamingMode()
                .build();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env, settings);
```

我们使用的是Blink Planner并且是stream模式。


###### Table API使用
我们可以将一个DataStream转换成Table

```java
Table studentTable = tableEnv.fromDataStream(studentStream);
```

转换成Table之后, 我们可以使用相关API进行选取相应的字段和过滤条件

```java
Table studentTableResult = studentTable.select($("name"), $("score"))
                .where("score >= 60");
```

比如我要查看分数大于等于60分的学生姓名和实际分数, 可以通过select和where两个api完成, 是不是和sql很像只不过变成了API。

###### SQL 使用
要想使用SQL则需要将数据注册成一张表, 这里我们可以注册一张临时的视图进行查询, 视图数据可以是Table也可以是DataStream。

```java
tableEnv.createTemporaryView("student", studentTable);
```
我们将刚才的Table注册为一个临时视图, 可以用于查询, 表的名字就是student

有了视图之后就和写sql没有任何的区别, 需求和上面一样
```java
String sql = "select name, score from student where score >= 60";
```

查询sql并返回对应数据
```java
Table studentSQLTableResult = tableEnv.sqlQuery(sql);
```
可以看到返回的也是一个Table类型。

但是Table类型是不可以直接打印数据的, 只能打印结构类型, 即schema, 如果想要输出可以将其转换成DataStream进行输出

```java
tableEnv.toAppendStream(studentTableResult, Row.class).print("studentTable");
tableEnv.toAppendStream(studentSQLTableResult, Row.class).print("studentSQL");
```


以上就是一个完整的Table API & SQL的程序。