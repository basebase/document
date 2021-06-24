### flink入门-代码实现

#### 水位线设置
前面, 我们已经把一些基础理论了解了, 下面就需要通过代码来进行实践, 如果设置我们的时间戳以及水位线生成。

水位线的生成DataStream可以通过三种模式完成:
1. 在数据源完成, 利用SourceFunction在应用读入数据流的时候分配时间戳和生成水位线, 比如我们在读取kafka的数据源时就可以使用该种模式。
2. 周期分配器(periodic assigner), DataStream提供了一个AssignerWithPeriodicWatermarks的用户自定义函数, 他可以用来从每条记录提取时间戳, 并周期性的响应获取当前水位线的查询请求。
3. 定点分配器, DataStream还提供了一个AssignerWithPunctuatedWatermarks用户自定义函数, 可用于需要根据特殊输入记录生成水位线的情况。


在使用WaterMark之前呢, 我们还需要注意的一个问题点就是, 必须将事件语义修改为***事件时间***否则是没有效果的

```java
// 使用事件时间
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

设置完时间特性为EventTime后就可以对时间戳和水印进行处理, 从而实现事件时间相关操作, 当然, 在使用EventTime的同时, 仍然可以使用处理时间, 只需要设置ProcessingTime即可

```java
env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime)
```


##### 周期性水位线分配器
周期性分配水位线的含义是我们会指示系统以固定的机器时间间隔来发出水位线并推动事件时间前进。默认的时间间隔为200毫秒, 我们也可以进行修改
```java
env.getConfig().setAutoWatermarkInterval(5000);    // 设置5秒生成一次watermark
```
设置完成后, flink会每5秒调用一次AssignerWithPeriodicWatermarks中的getCurrentWatermark()方法, 如果该方法的返回值非空并且它的时间戳大于上一次水位线的时间戳, 那么算子就会发出一个新的水位线。这项检查对于保证事件时间持续递增十分必要, 一旦检查失败将不会生成水位线。


下面展示自定义一个周期水位线
```java
orderOperator.assignTimestampsAndWatermarks(new AssignerWithPeriodicWatermarks<Order>() {
    private long bound = 3000;      // 3s
    private long maxTs = Long.MIN_VALUE;    // 最大时间戳

    @Nullable
    @Override
    public Watermark getCurrentWatermark() {
        // 生成一个具有3s延迟的水位线
        return new Watermark(maxTs - bound);
    }

    @Override
    public long extractTimestamp(Order element, long recordTimestamp) {
        try {
            long currentTime = DateUtils.strToTimestamp(element.getCreatetime());
            // 更新最大时间戳
            maxTs = Math.max(currentTime, maxTs);
            return currentTime;
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return 0;
    }
});
```


通常, 我们无需自定义周期性水位线生成, DataStream内置了两种周期性水位线时间戳分配器, 如果我们输入的元素时间戳是单调递增的(即不会出现乱序), 则可以使用一个简便方法
AscendingTimestampExtractor

```java
orderOperator.assignTimestampsAndWatermarks(new AscendingTimestampExtractor<Order>() {
    @Override
    public long extractAscendingTimestamp(Order element) {
        try {
            long createtime = DateUtils.strToTimestamp(element.getCreatetime());
            return createtime;
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return 0;
    }
});
```

另一个分配器则是比较常见的情况, 我们知道输入肯定会存在延迟, 针对这种情况, 可以使用BoundedOutOfOrdernessTimestampExtractor, 该类接收一个时间参数, 表示可以等待多长迟到数据

```java
orderOperator.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<Order>(Time.seconds(3)) {   // 等待3s
    @Override
    public long extractTimestamp(Order element) {
        try {
            long createtime = DateUtils.strToTimestamp(element.getCreatetime());
            return createtime;
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return 0;
    }
});
```

上面代码元素最多允许延迟3秒, 如果在3秒内到达可以进入对应窗口计算, 否则窗口已经计算完成并关闭, 对于这种迟到数据还需要额外处理。


##### 定点水位线分配器
有时候输入流中包含一些特殊的标记, 我们需要碰到此类标记才生成水位线, 可以使用AssignerWithPunctuatedWatermarks接口, 该接口中的checkAndGetNextWatermark()方法会在针对每个事件的extractTimestamp()方法后立即调用, 它可以决定是否生成一个新的水位线。

如果该方法返回一个非空并且大于之前值的水位线, 算子就会将这个新水位线发出。



#### flink1.11新版水位线生成
上面的水位线生成方式都是1.11之前的, 并且官方也不推荐使用上述API, 虽然API过期了, 但概念还是上面那么三个概念, 只不过调用有所变动

算子通过assignTimestampsAndWatermarks()方法来生成一个水位线, 传入是一个WatermarkStrategy实例, 不过WatermarkStrategy接口中有很多默认方法可以调用

```java
// 最大延迟等待3s
WatermarkStrategy.<Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))
// 单调递增
WatermarkStrategy.<Order>forMonotonousTimestamps();
```

上面两个方法是不是看着很熟悉, 我们在来看看上面两个方法的底层实现

```java

static <T> WatermarkStrategy<T> forBoundedOutOfOrderness(Duration maxOutOfOrderness) {
	return (ctx) -> new BoundedOutOfOrdernessWatermarks<>(maxOutOfOrderness);
}
static <T> WatermarkStrategy<T> forMonotonousTimestamps() {
	return (ctx) -> new AscendingTimestampsWatermarks<>();
}
```
不过新版水位线中, 还提供了一个非常贴心的功能***withIdleness***方法, 比如我们在读取kafka多个分区数据, 某一个分区在一段时间内未发送事件数据
则意味着 WatermarkGenerator 也不会获得任何新数据去生成 watermark。我们称这类数据源为空闲输入或空闲源。在这种情况下，当某些其他分区仍然发送事件数据的时候就会出现问题。由于下游算子 watermark 的计算方式是取所有不同的上游并行数据源 watermark 的最小值，则其 watermark 将不会发生变化。

withIdleness方法就是解决此问题。

```java

WatermarkStrategy<Order> orderWatermarkStrategy = WatermarkStrategy.<Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))
.withIdleness(Duration.ofMillis(1))     // 等待1分钟
.withTimestampAssigner(new SerializableTimestampAssigner<Order>() {
    @Override
    public long extractTimestamp(Order element, long recordTimestamp) {
        try {
            long currentTimeStamp = DateUtils.strToTimestamp(element.getCreatetime());
            return currentTimeStamp;
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return 0;
    }
});
```