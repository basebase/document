### flink入门-filter算子

#### 实战
filter算子作为过滤算子, 将不需要的数据进行过滤, 传入N个球形状态, 要满足对应面积才进入后续计算, 否则该数据被丢弃;

![filter算子](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/transformation/filter%E7%AE%97%E5%AD%90.png?raw=true)

```java
public class FilterOperator {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        String inPath = FileSource.class.getClassLoader().getResource("student").getPath();
        // 读取文件数据作为数据源
        DataStreamSource<String> fileSource = env.readTextFile(inPath);

        SingleOutputStreamOperator<String> filterOperator = fileSource.filter(new FilterFunction<String>() {
            @Override
            public boolean filter(String s) throws Exception {
                return s.length() >= 10;
            }
        });

        filterOperator.print();
        env.execute("Filter Operator Test Job");
    }
}

```