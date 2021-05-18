### flink入门-union算子

#### 实战
union算子可以合并多条流, 但是多条流的数据类型必须一致, 这个就和connect算子合并流有点不同, connect可以不限制数据类型但只能两条流关联;

使用union算子后数据类型依然是DataStream不会被改变

```java
public class UnionOperator {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        // 1. 第一条流数据
        DataStreamSource<Tuple2<String, Integer>> source1 = env.fromCollection(Arrays.asList(
                new Tuple2<>("张三", 100),
                new Tuple2<>("李四", 40),
                new Tuple2<>("王五", 10),
                new Tuple2<>("赵柳", 900)
        ));

        DataStreamSource<Tuple2<String, Integer>> source2 = env.fromCollection(Arrays.asList(
                new Tuple2<>("奎因", 70),
                new Tuple2<>("路费", 88),
                new Tuple2<>("山治", 7)
        ));

        DataStreamSource<Tuple2<String, Integer>> source3 = env.fromCollection(Arrays.asList(
                new Tuple2<>("历飞雨", 1),
                new Tuple2<>("韩立", 2),
                new Tuple2<>("大头怪人", 3)
        ));


        // source1调用union方法, 泛型已经被限制为了Tuple2<String, Integer>所以无法union source4
        DataStreamSource<Integer> source4 = env.fromCollection(Arrays.asList(
                900, 1000, 1100, 1200
        ));

        DataStream<Tuple2<String, Integer>> unionStream = source1.union(source2, source3);
        unionStream.print();

        env.execute("Union Operator Test Job");
    }
}
```