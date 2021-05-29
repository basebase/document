### flink入门-kafka数据源

#### 实战
在学习source算子的时候我们已经读取过kafka的数据源, 所以不再需要添加相关依赖, 现在我们需要将计算过后的数据写入到kafka中方便后面的程序去使用;

在消费kafka数据我们使用的是***FlinkKafkaConsumer类***, 但是在写入的时候我们需要关注的则是***FlinkKafkaProducer类***

我们先将字符串数据进行发送
```java

public class KafkaSink {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> socketSource =
                env.socketTextStream("localhost", 8888);

        Properties kafkaConf = new Properties();
        kafkaConf.setProperty("bootstrap.servers", "localhost:9092");

        /**
         *      写入数据到kafka使用FlinkKafkaProducer
         */
        FlinkKafkaProducer<String> kafkaSink =
                new FlinkKafkaProducer<>("kafka_sink", new SimpleStringSchema(), kafkaConf);

        SingleOutputStreamOperator<String> strStream = socketSource.filter(value -> value != null && value.length() > 0).map(value -> {
            return value.replaceAll(",", "").toUpperCase();
        });

        strStream.addSink(kafkaSink);
        env.execute("Kafka Sink Test Job");
    }
}
```