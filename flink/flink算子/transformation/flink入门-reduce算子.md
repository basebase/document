### flink入门-reduce算子

#### 实战
在前面介绍keyBy算子时提到过reduce操作, 前面我们已经使用max,maxBy算子, 但是这两个算子并不能满足聚合需求, 比如我想用名字长的替换短的并且分数累加, 这个时候就可以使用到reduce算子, reduce算子也可以实现max和maxBy同等功能, 如果深入源码会发现其底层最后也是使用reduce来实现的。

reduce算子是分组聚合计算比较通用的方法, 当普通算子无法满足可以通过reduce来实现;



先来看看要实现reduce算子, 需要实现ReduceFunction, 其中有一个reduce方法, 里面有两个参数:  
参数1: 上一次结果数据  
参数2: stream最新的数据  
返回参数1和参数2合并后的数据, 本质上参数1就是返回的中间结果数据  

第一次调用reduce方法时value1的值为stream的第一个元素, 原理其实和java reduce差不多

```java
public interface ReduceFunction<T> extends Function, Serializable {
    T reduce(T value1, T value2) throws Exception;
}
```

```java
public class ReduceOperator {
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

        mapOperator.keyBy("id")
                .reduce(new ReduceFunction<Student>() {
                    // 不同key下num是不共享的
                    private int num = 0;
                    @Override
                    public Student reduce(Student value1, Student value2) throws Exception {
                        num += 1;
                        System.out.println("value1: " + value1 + " value2: " + value2);
                        System.out.println("key(" + value1.getId() + ")调用次数: " + num);
                        // 使用最新数据的名称替代旧的名称, 并且累加所有分数
                        return new Student(value1.getId(), value2.getName(), value2.getScore() + value1.getScore());
                    }
                }).print();

        env.execute("Reduce Operator Test Job");
    }
}
```


