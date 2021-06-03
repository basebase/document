### flink入门-kafka目标源

#### 实战
在学习source算子的时候我们已经读取过kafka的数据源, 所以不再需要添加相关依赖, 现在我们需要将计算过后的数据写入到kafka中方便后面的程序去使用;

在消费kafka数据我们使用的是***FlinkKafkaConsumer类***, 但是在写入的时候我们需要关注的则是***FlinkKafkaProducer类***

我们先以字符串数据进行发送
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

字符串数据是最简单的数据, 线上业务通常都是一个java bean对象, 这个时候需要序列化对象则需要自己实现接口, 我么可以利用***ObjectMapper***类实现序列化

```java
public class KafkaSink {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> socketSource =
                env.socketTextStream("localhost", 8888);

        Properties kafkaConf = new Properties();
        kafkaConf.setProperty("bootstrap.servers", "localhost:9092");

        FlinkKafkaProducer<Student> studentKafkaSink =
                new FlinkKafkaProducer<>("kafka_sink", new StudentSerializationSchema(), kafkaConf);

        DataStream<Student> studentStream = socketSource.filter(value -> value != null && value.length() > 0).map(value -> {
            String[] fields = value.split(",");
            return new Student(Integer.parseInt(fields[0]), fields[1], Double.parseDouble(fields[2]));
        }).returns(Types.POJO(Student.class));


        studentStream.print();

        // 添加输出目标
        studentStream.addSink(studentKafkaSink);

        env.execute("Kafka Sink Test Job");
    }

    private static class StudentSerializationSchema implements SerializationSchema<Student> {
        ObjectMapper mapper;

        @Override
        public byte[] serialize(Student element) {
            byte[] b = null;
            if (mapper == null) {
                mapper = new ObjectMapper();
            }
            try {
                b= mapper.writeValueAsBytes(element);
            } catch (JsonProcessingException e) {
                e.printStackTrace();
            }
            return b;
        }
    }
}
```

#### 参考
[How to implement FlinkKafkaProducer serializer for Kafka 2.2](https://stackoverflow.com/questions/58644549/how-to-implement-flinkkafkaproducer-serializer-for-kafka-2-2)

[Flink的sink实战之二：kafka](https://blog.csdn.net/boling_cavalry/article/details/105598224)
