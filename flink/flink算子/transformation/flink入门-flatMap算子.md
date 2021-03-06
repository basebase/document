### flink入门-flatMap算子

#### 实战
flatMap算子可以理解一对多的转换关系, 比如我们只传入一个球形状态可以根据需要生成多个三角形, 当然也可以是一个三角形;

![flatMap算子](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/transformation/flatMap%E7%AE%97%E5%AD%90.png?raw=true)

```java
public class FlatMapOperator {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();


        String inPath = FileSource.class.getClassLoader().getResource("student").getPath();
        // 读取文件数据作为数据源
        DataStreamSource<String> fileSource = env.readTextFile(inPath);

        /***
         * 和map方法类似，只不过flatMap可以将一个对象包装成更多的对象发送出去
         */
        SingleOutputStreamOperator<String> flatMapOperator = fileSource.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String data, Collector<String> collector) throws Exception {
                String[] values = data.split(",");
                for (String value : values) {
                    collector.collect(value);
                }
            }
        });

        flatMapOperator.print();

        env.execute("FlatMap Operator Test Job");
    }
}
```