### flink入门-map算子

#### 实战
flink map算子我们可以将其理解为一个一对一的类型转换, 可以参考下图
![map算子](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/transformation/map%E7%AE%97%E5%AD%90.png?raw=true)

我们传入一个圆形可以获得等价数量的三角形, 当然也可以是圆形或者其它形状;

```java
public class MapOperator {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        String inPath = FileSource.class.getClassLoader().getResource("student").getPath();
        // 读取文件数据作为数据源
        DataStreamSource<String> fileSource = env.readTextFile(inPath);

        /***
         *      MapFunction中的泛型T和O分别代表为:
         *          T: 调用map算子的泛型类型，也就是String，可以看成是我们的输入类型
         *          O: 通过map算子转换后的类型，也就是Integer，可以看成是输出类型
         *
         */
        SingleOutputStreamOperator<Integer> mapOperator = fileSource.map(new MapFunction<String, Integer>() {
            @Override
            public Integer map(String s) throws Exception {
                return s.length();
            }
        });

        mapOperator.print();

        env.execute("Map Operator Test Job");
    }
}
```