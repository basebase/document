### flink入门-集合数据源

#### 实战

使用集合作为数据源, 我们有两个点注意:
1. 数据条数固定, 无法动态增加数据
2. 数据类型固定, 否则抛出异常

```java
public class CollectionSource {
    public static void main(String[] args) throws Exception {
        // 1. 创建执行环境
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        // 2. 创建一个学生集合数据源
        DataStreamSource<Student> collectionSource = env.fromCollection(Arrays.asList(
                new Student(1, "A", 111.1),
                new Student(2, "B", 222.2),
                new Student(3, "C", 333.3),
                new Student(4, "D", 444.4),
                new Student(5, "E", 555.5)
        ));

        // 3. 创建一个数字集合数据源
        DataStreamSource<Integer> integerDataStreamSource = env.fromElements(1, 2, 3);
        // 报错 java.lang.IllegalArgumentException: The elements in the collection are not all subclasses of java.lang.Integer
        // DataStreamSource<? extends Serializable> dataStreamSource = env.fromElements(1, "2", 3);


        // 4. 打印输出
        collectionSource.print("collection source");
        integerDataStreamSource.print("int source");


        // 5. 执行
        env.execute("Collection Source Test Job");
    }
}
```