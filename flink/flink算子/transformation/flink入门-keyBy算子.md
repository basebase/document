### flink入门-map算子

#### 实战
keyBy算子通常不会对数据的转换, 而是将数据划分到不同的分区中, 每个相同key都在同一个分区中, 但是一个分区中不一定只包含一种key数据, 可能会存在多个key在同一分区内的情况, 如: 对key进行hash后取模计算

![keyBy算子](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/transformation/keyBy%E7%AE%97%E5%AD%90.png?raw=true)

在使用keyBy算子之前我们是无法使用聚合滚动方法的, 常见的如: reduce, sum, max等操作, 在算子进行分组后数据类型由DataStream转换为KeydStream


算子在进行keyBy通常有下面几个选项:
```java
// 适用使用Tuple类型数据进行keyBy, 返回的Key值也是Tuple
public KeyedStream<T, Tuple> keyBy(int... fields)
// 使用使用java bean对象类型数据进行keyBy, 返回Key值也是Tuple
public KeyedStream<T, Tuple> keyBy(String... fields)
// 适用上面两种类型, 可定义Key返回类型
public <K> KeyedStream<T, K> keyBy(KeySelector<T, K> key)
```

```java
public class KeydOperator {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        String inPath = FileSource.class.getClassLoader().getResource("student").getPath();
        // 读取文件数据作为数据源
        DataStreamSource<String> fileSource = env.readTextFile(inPath);


        // 1. 处理数据, 转换为Student对象
        DataStream<Student> mapOperator = fileSource.map(new MapFunction<String, Student>() {
            @Override
            public Student map(String value) throws Exception {
                String[] fileds = value.split(",");
                return new Student(Integer.parseInt(fileds[0]), fileds[1], Double.parseDouble(fileds[2]));
            }
        });

        // 2. 想要对算子做一些聚合计算, 需要进行keyBy分组
        // 2.1 数据类型从DataStream转为KeydStream, 不过本质还是一个DataStream
        KeyedStream<Student, Tuple> studentKeyedStream = mapOperator.keyBy("id");

        /**
         *     max的使用:
         *      相同key数据如果当前数据比上一条数据的score值大, 则替换score值, 其余数据不替换, 即: 只更新max中的字段值, 其它数据则以第一条进入的数据为标准
         *
         *     maxBy使用:
         *      相同key数据如果当前数据比上一条数据的score值大, 则使用当前一整条数据替换, 即: 当前数据是最大score全部内容数据
         */
        studentKeyedStream.max("score").print();
        studentKeyedStream.maxBy("score").print();

        env.execute("Keyd Operator Test Job");
    }
}
```