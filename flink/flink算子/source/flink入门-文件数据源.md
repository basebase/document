### flink入门-文件数据源

#### 实战
读取文件数据源要优于集合数据源, 首先代码和数据源是分开的, 不需要新增数据源进而修改代码, 并且数据类型都是字符串类型
不在限定为某一个类型, 如果需要转换为其它类型则后面的算子部分会进行介绍;


```java
public class FileSource {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();


        String inPath = FileSource.class.getClassLoader().getResource("student").getPath();
        // 读取文件数据作为数据源
        DataStreamSource<String> fileSource = env.readTextFile(inPath);

        fileSource.print("file source");

        env.execute("File Source Test Job");
    }
}
```