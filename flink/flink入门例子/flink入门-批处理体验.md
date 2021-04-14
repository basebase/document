### flink入门-批处理体验

#### 快速上手
经过前面的项目初始化, 现在我们可以正式开始编写第一个flink程序, 通过使用flink的批处理来实现一个WordCount程序;  
在编写程序之前, 我们应该先准备好一个要被处理的数据, 这个大家自行准备即可;


#### 创建运行时环境
执行flink程序, 我们需要创建一个运行环境, 这是执行一切flink程序的基础, 只有创建运行环境之后才能执行我们flink程序, 对于如何创建运行环境其实非常简单

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
```

#### 读取数据源
有了运行环境之后, 那么现在最重要的就是如何读取我们的数据, flink提供了很多API来帮助我们读取所需要的数据源, 不过我这里创建的是
一个文本所以选择readText, 当然也支持csv等文件类型

```java
String inPath = WrodCountBatch.class.getClassLoader().getResource("wordcount").getPath();
// DataSource<String> dataSource = env.readTextFile("");
DataSet<String> source = env.readTextFile(inPath);
```

当我们读取数据之后返回的是一个DataSource产生一个数据源, 但是当我们深入到DataSource内部后发现其底层就是一个DataSet类
我们本质上操作的还是一个DataSet, 所以flink批处理API也被称为DataSet API

#### 数据切分
数据已经读取进来了, 现在我们则需要处理读取进来的数据, 我们需要实现一个接口FlatMapFunction用于对数据切分, 实现该接口的方法很简单, 就是将读取的数据按照分隔符切分并形成一个二元组

```java

public static class WordCountFlatMapFunction implements FlatMapFunction<String, Tuple2<String, Integer>> {
    @Override
    public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
        if (s == null || s.length() == 0)
            return;
        String[] tokens = s.split(",");
        for (String token : tokens) {
            collector.collect(new Tuple2<>(token, 1));      // 每个单词都固定是1
        }
    }
}
```

这里需要注意的是Tuple2这个类, 我们要导入的是org.apache.flink.api.java.tuple.Tuple2而不是scala的包


#### 数据分组统计
数据现在已经被切分开了, 但是每个单独都是固定的数量1, 我们需要对其进行分组并累加, 这个时候则需要使用到groupBy和sum方法了, 
对于groupBy来说, 我们可以使用三种方式来实现

1. 实现KeySelector
2. 传入位置
3. 传入字段名称

但我们使用的是一个二元组的形式并不清楚字段名称也不打算实现KeySelector接口, 所以采用第二种方式传入一个具体位置(当然可以传入多个);

对于传入位置是从0开始的, 而我们是一个二元组所以最多是0到1两个值, 如果传入的是0则按照二元组第一个值进行分组, 传入多个位置就是多个值进行分组

了解groupBy之后对于sum也是一样的是, 按照二元组第二个值进行聚合

```java
DataSet<Tuple2<String, Integer>> resultSet = source.flatMap(new WordCountFlatMapFunction())
                .groupBy(0)
                .sum(1);

resultSet.print();      // 打印输出
```

![输出结果](https://github.com/basebase/document/blob/master/flink/image/%E6%89%B9%E5%A4%84%E7%90%86%E4%BD%93%E9%AA%8C/%E8%BE%93%E5%87%BA%E7%BB%93%E6%9E%9C.png?raw=true)
