### flink入门-kafka数据源

#### 实战

对于线上数据服务来说, 通常我们都是从消息队列中读取数据, 所以我们以kafka作为数据源进行读取, 使用kafka为数据源则需要添加对应依赖, maven中添加如下内容

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
```

添加依赖后, 我们则需要关注一个类 ***FlinkKafkaConsumer*** 该类作为消费端, 用于读取kafka数据, 其中有三个参数需要用户传入;

1. kafka topic名称
2. 反序列化kafka数据类型
3. kafka consumer属性配置


```java
public class KafkaSource {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        // 添加kafka配置
        Properties kafkaPorps = new Properties();
        // kafka集群
        kafkaPorps.setProperty("bootstrap.servers", "localhost:9092");
        // kafka消费组
        kafkaPorps.setProperty("group.id", "test");

        FlinkKafkaConsumer<String> kafkaConsumer =
                // 参数1: kafka topic
                // 参数2: 反序列化kafka数据类型
                // 参数3: kafka参数
                new FlinkKafkaConsumer<>("test_topic", new SimpleStringSchema(), kafkaPorps);

        // 通过addSource添加数据源
        DataStreamSource<String> kafkaSource = env.addSource(kafkaConsumer);

        kafkaSource.print("kafka source");
        env.execute("Kafka Source Test Job");
    }
}
```