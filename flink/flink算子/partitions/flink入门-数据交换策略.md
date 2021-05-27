### flink入门-数据交换策略

数据交换策略定义了如何将数据分配给不同任务, 在使用DataStream API创建程序时, 系统会根据操作语义和配置的并行度自动选择数据分区策略并将数据转发到正确的目标中。但某些时候, 我们希望能够在应用级别控制这些分区策略或者自定义分区器。

例如: 如果我们知道DataStream的并行分区存在数据倾斜现象, 那么可能就希望通过重新平衡数据来均匀分配后续算子的负载。又或许我们想按照自有逻辑定义策略分发时间等；

在flink中有八种不同的交换策略, 也被称为分区器(数据分区), 如下图:

![数据分区-类关系图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/%E6%95%B0%E6%8D%AE%E5%88%86%E5%8C%BA-%E7%B1%BB%E5%85%B3%E7%B3%BB%E5%9B%BE.png?raw=true)

下面八个子类都继承实现自StreamPartitioner类, StreamPartitioner类实现ChannelSelector接口, 这里有非常有必要介绍ChannelSelector接口。

```java
public interface ChannelSelector<T extends IOReadableWritable> {
    // 初始化channels数量, channels可以理解为下游算子(Operator)的并行数(sub-task)
	void setup(int numberOfChannels);
    // 每个分区都会实现该方法用来确定将数据发送到下游算子哪个实例(sub-task)中, 在[0-numberOfChannels)之间选取
	int selectChannel(T record);
    // 数据是否广播, 除了broadcast策略为true其余分区默认都是false
	boolean isBroadcast();
}
```

#### 实战

##### 1. shuffle随机策略
我们可以通过DataStream.shuffle()方法实现随机数据交换策略, 该方法会依照均匀分布随机将数据发送下游算子并行任务中

###### shuffle分区源码
```java
public class ShufflePartitioner<T> extends StreamPartitioner<T> {
	private Random random = new Random();
	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
        // 产生[0, numberOfChannels)伪随机数, 随机发送到下游算子的sub-task中, 不过是均匀分布, 每个sub-task都差不多数量
		return random.nextInt(numberOfChannels);
	}
}
```

###### shuffle随机分区例子
```java
DataStreamSource<String> socketSource = env.socketTextStream("localhost", 8888);       
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
        .shuffle()
        .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(3)
        .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2).shuffle()
        .map(value -> value.getBytes().length).name("strSize").setParallelism(1).print();
```

观察到replaceMap算子向upStrMap算子传输数据策略是使用shuffle了, 并且数据分布相对均匀

![shuffle分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/shuffle%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![shuffle分区算子运行图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/shuffle%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE.png?raw=true)


#### 2. rebalance轮询策略
通过DataStream.rebalance()方法实现轮询数据交换策略, 该方法会将数据轮询方式均匀分配给下游算子任务

###### rebalance分区源码
```java
public class RebalancePartitioner<T> extends StreamPartitioner<T> {
	private int nextChannelToSendTo;
	@Override
	public void setup(int numberOfChannels) {
		super.setup(numberOfChannels);
        // 初始channel值, 返回[0, numberOfChannels)的伪随机数
		nextChannelToSendTo = ThreadLocalRandom.current().nextInt(numberOfChannels);
	}

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
        // 循环发送到下游task, 这里使用取模运算实现的循环
        // 假设: nextChannelToSendTo为1, numberOfChannels(下游算子并行数为3)
        // 第一次发送sub-task-id为: (1 + 1) % 3 = sub-task-2的任务
        // 第二次发送sub-task-id为: (2 + 1) % 3 = sub-task-0的任务
        // 依次类推...
		nextChannelToSendTo = (nextChannelToSendTo + 1) % numberOfChannels;
		return nextChannelToSendTo;
	}
}
```

###### rebalance轮询分区例子
```java
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
            .rebalance()
            .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(3)
            .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2).rebalance()
            .map(value -> value.getBytes().length).name("strSize").setParallelism(1).print();
```

观察到replaceMap算子向upStrMap算子传输数据策略是使用rebalance了, 并且数据是轮询发送到每个实例中

![rebalance分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/rebalance%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![rebalance分区算子运行图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/rebalance%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE.png?raw=true)


#### 3. rescale重调策略
我们可以通过DataStream.rescale()方法实现重调数据交换策略, 该方法也会依照轮询的方式把数据发送到下游算子实例中, 但是分发的目标仅限部分实例, 如何理解呢?

假设: 上游算子并行度为2, 下游算子并行度为4, 则上游算子第一个实例(sub-task-0)以循环的方式将记录输出到下游的两个并行度上, 上游第二个实例(sub-task-1)以循环的方式将记录输出到下游另外两个并行度上;

如果, 上游算子并行度为4, 下游算子并行度为2, 则上游两个并行度数据输出到下游一个并行度上, 上游另外两个并行度数据输出到下游另外一个并行度上;

rebalance策略则会和所有任务进行通信并发送数据, 而rescale只会和下游部分算子实例建立通信

###### rescale分区源码
```java

public class RescalePartitioner<T> extends StreamPartitioner<T> {
	private int nextChannelToSendTo = -1;

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		if (++nextChannelToSendTo >= numberOfChannels) {
			nextChannelToSendTo = 0;
		}
		return nextChannelToSendTo;
	}
}
```

###### rescale重调分区例子
```java
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
            .rescale()
            .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(4)
            .rescale()
            .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2)
            .map(value -> value.getBytes().length).name("strSize").setParallelism(1).print();
```

观察到replaceMap算子向upStrMap算子传输数据策略是使用rebalance了, 上游算子(sub-task-0)发送数据到下游算子(sub-task-0, sub-task-1), 上游算子(sub-task-1)发送数据到下游算子(sub-task-2, sub-task-3)

![rescale分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/rescale%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![rescale分区算子运行图1](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/rescale%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE1.png?raw=true)

![rescale分区算子运行图2](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/rescale%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE2.png?raw=true)


#### 4. broadcast广播策略
通过DataStream.broadcast()方法实现广播数据交换策略, 该方法会将数据复制并发送到所有下游算子并行任务中

###### broadcast分区源码
```java
public class BroadcastPartitioner<T> extends StreamPartitioner<T> {
	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
        // Broadcast模式是直接发送到下游的所有task，所以不需要通过下面的方法选择发送的通道
		throw new UnsupportedOperationException("Broadcast partitioner does not support select channels.");
	}

	@Override
	public boolean isBroadcast() {
		return true;
	}
}
```

###### broadcast广播分区例子
```java
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
            .broadcast()
            .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(4)
            .broadcast()
            .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2)
            .map(value -> value.getBytes().length).name("strSize").setParallelism(1).print();

```
观察到replaceMap算子向upStrMap算子传输数据策略是使用broadcast了, 上游有两个并行任务, 下游有4个并行任务, 假设上游算子任务sub-task-0和sub-task-1各5条数据, 都需要复制到下游算子任务中, 下游4个并行任务, 这样每个并行任务都会有10条重复数据, 然后继续发送到下游算子中, 平白多出很多重复数据



![broadcast分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/broadcast%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![broadcast分区算子运行图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/broadcast%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE.png?raw=true)


#### 5. global全局策略
通过DataStream.global()方法实现全局数据交换策略, 该方法会将数据发送到下游算子第一个并行任务中

###### global分区源码
```java
public class GlobalPartitioner<T> extends StreamPartitioner<T> {
	private static final long serialVersionUID = 1L;

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
        // 返回0表示只发送给下游算子SubTask Id=0的子任务上
		return 0;
	}
}
```

###### global全局分区例子
```java
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
            .global()
            .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(4)
            .global()
            .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2)
            .map(value -> value.getBytes().length).name("strSize").setParallelism(1).print();
```

观察到replaceMap算子向upStrMap算子传输数据策略是使用global了, 上游有两个并行任务, 下游有4个并行任务, 所有数据都发送到下游第一个并行任务中, 其它任务没有数据


![global分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/global%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![global分区算子运行图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/global%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE.png?raw=true)


#### 6. forward转发策略
通过DataStream.forward()方法实现转发数据交换策略, 上游算子和下游算子任务并行度是一致的, 即1:1进行传输

###### forward分区源码
```java
public class ForwardPartitioner<T> extends StreamPartitioner<T> {
	private static final long serialVersionUID = 1L;

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		return 0;
	}
}
```

###### forward转发分区例子
```java
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
            .forward()
            .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(2)
            .forward()
            .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2).disableChaining()
            .map(value -> value.getBytes().length).name("strSize").setParallelism(2).print();
```
观察到replaceMap算子向upStrMap算子传输数据策略是使用forward了, 上游有两个并行任务, 下游有2个并行任务, 上游第一个并行任务有几条数据下游第一个并行任务就有几条

***提示1:*** 这里需要注意, 如果上下游并行任务不一致则会抛出异常 
```text
Exception in thread "main" java.lang.UnsupportedOperationException: Forward partitioning does not allow change of parallelism. Upstream operation:...
```

***提示2:*** 如果上下游算子没有指定分区器的情况下, 上下游算子并行度一致, 则使用forward分区, 否则使用rescale分区

![forward分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/forward%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![forward分区算子运行图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/forward%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE.png?raw=true)

#### 7. key分组策略
通过DataStream.key()方法实现基于键值的数据交换策略, 根据键值对数据进行分区, 保证相同键值数据一定会交由同一个任务处理

###### key分区源码
```java
public class KeyGroupStreamPartitioner<T, K> extends StreamPartitioner<T> implements ConfigurableStreamPartitioner {
	private final KeySelector<T, K> keySelector;
	private int maxParallelism;
	public KeyGroupStreamPartitioner(KeySelector<T, K> keySelector, int maxParallelism) {
		Preconditions.checkArgument(maxParallelism > 0, "Number of key-groups must be > 0!");
		this.keySelector = Preconditions.checkNotNull(keySelector);
		this.maxParallelism = maxParallelism;
	}

	@Override
	public int selectChannel(SerializationDelegate<StreamRecord<T>> record) {
		K key;
		try {
			key = keySelector.getKey(record.getInstance().getValue());
		} catch (Exception e) {
			throw new RuntimeException("Could not extract key from " + record.getInstance().getValue(), e);
		}
		return KeyGroupRangeAssignment.assignKeyToParallelOperator(key, maxParallelism, numberOfChannels);
	}
}


public final class KeyGroupRangeAssignment {

    // 根据key分配一个并行算子实例的索引，该索引即为该key要发送的下游算子实例的路由信息, 即该key发送到哪一个task
	public static int assignKeyToParallelOperator(Object key, int maxParallelism, int parallelism) {
		Preconditions.checkNotNull(key, "Assigned key must not be null!");
		return computeOperatorIndexForKeyGroup(maxParallelism, parallelism, assignToKeyGroup(key, maxParallelism));
	}

    // 根据key分配一个分组id(keyGroupId)
	public static int assignToKeyGroup(Object key, int maxParallelism) {
		Preconditions.checkNotNull(key, "Assigned key must not be null!");
		return computeKeyGroupForKeyHash(key.hashCode(), maxParallelism);
	}

    // 根据key分配一个分组id(keyGroupId),
	public static int computeKeyGroupForKeyHash(int keyHash, int maxParallelism) {
        // 与maxParallelism取余，获取keyGroupId
		return MathUtils.murmurHash(keyHash) % maxParallelism;
	}

    // 计算分区index，即该key group应该发送到下游的哪一个算子实例
	public static int computeOperatorIndexForKeyGroup(int maxParallelism, int parallelism, int keyGroupId) {
		return keyGroupId * parallelism / maxParallelism;
	}
}
```

###### key分区例子
```java
socketSource.map(value -> value.replaceAll(",", "")).name("replaceMap").setParallelism(2)
            .keyBy(value -> value)
            .map(value -> value.toUpperCase()).name("upStrMap").setParallelism(4)
            .keyBy(value -> value)
            .map(value -> value.toLowerCase()).name("lowMap").setParallelism(2)
            .map(value -> value.getBytes().length).name("strSize").setParallelism(1).print();
```

观察到replaceMap算子向upStrMap算子传输数据策略是使用hash(key分组)了, 上游有两个并行任务, 下游有2个并行任务, 上游相同key数据都发送到下游同一个任务中, 但是一个任务中并不会仅仅包含一种key数据

![key分区算子分区逻辑图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/key%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E5%88%86%E5%8C%BA%E9%80%BB%E8%BE%91%E5%9B%BE.png?raw=true)

![key分区算子运行图](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90/partitions/key%E5%88%86%E5%8C%BA%E7%AE%97%E5%AD%90%E8%BF%90%E8%A1%8C%E5%9B%BE.png?raw=true)



#### 8. 自定义分组策略
通过DataStream.partitionCustom()方法实现自定义数据交换策略, 如果上面分区策略无法满足业务要求则我们可以自己实现想要的分区策略, 只需要传入一个Partitioner实现以及KeySelector实现即可, 底层是调用CustomPartitionerWrapper类

###### 自定义分区例子
```java
public class DataFlowCustomPartitioners {

    public static void main(String[] args) throws Exception {
        socketSource.map(value -> {
            String[] fields = value.split(",");
            return new Student(Integer.parseInt(fields[0]), fields[1], Double.parseDouble(fields[2]));
        }).name("studentMap").setParallelism(2)
                .partitionCustom(new StudentPartitioner(), new StudentKeySelector())
                .map(value -> value.toString().toUpperCase()).name("upStrMap").setParallelism(4).print();    
        }

    // 根据学生ID判断将数据发送到哪个子任务中
    private static class StudentPartitioner implements Partitioner<Integer> {
        @Override
        public int partition(Integer key, int numPartitions) {
            if (key < 0) {
                return 0;
            } else if (key > 0 && key < 10) {
                return 1;
            } else if (key > 10 && key < 20) {
                return 2;
            }
            return 3;
        }
    }

    // 以学生ID作为key
    private static class StudentKeySelector implements KeySelector<Student, Integer> {
        @Override
        public Integer getKey(Student value) throws Exception {
            return value.getId();
        }
    }
}
```

#### 总结

|  类型   | 描述  |
|  ----  | ----  |
| global  | 全部发送到第一个sub-task |
| broadcast  | 发送有所sub-task |
| forward  | 一对一发送 |
| shuffle  | 随机均匀发送 |
| rebalance  | 轮询发送 |
| rescale  | 本地轮询发送 |
| partitionCustom  | 自定义 |
| key  | 相同key发送 |

#### 参考
[Flink的八种分区策略源码解读](https://jiamaoxiang.top/2020/03/30/Flink%E7%9A%84%E5%85%AB%E7%A7%8D%E5%88%86%E5%8C%BA%E7%AD%96%E7%95%A5%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB/#CustomPartitionerWrapper)

[Flink 数据交换策略Partitioner](http://smartsi.club/physical-partitioning-in-apache-flink.html)