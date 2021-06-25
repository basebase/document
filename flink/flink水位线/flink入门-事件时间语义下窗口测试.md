### flink入门-代码实现

#### 事件时间窗口测试
现在我们可以将时间设置为事件时间, 这样窗口使用的是事件时间而不在是系统时间。并且可以等待一定的延迟数据然后触发窗口进行计算。


下面, 我们要编写一个统计订单金额的程序


先将数据转为对应的POJO对象, 并设置水位线。并按照ID将数据进行分组, watermark是一个全局对象无需担心keyBy后会导致一个key一个watermark, 我们以5秒为一个窗口进行计算并且有3秒的数据延迟等待, 等watermark到达对应的window_end就会触发计算
```java
SingleOutputStreamOperator<Order> orderOperator = socketSource.filter(str -> !str.equals("") && str.split(",").length == 4)
                .map(str -> {
                    String[] fields = str.split(",");
                    return new Order(Integer.parseInt(fields[0]), fields[1], Integer.parseInt(fields[2]), fields[3]);
                });

WatermarkStrategy<Order> orderWatermarkStrategy = WatermarkStrategy.<Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))
        .withIdleness(Duration.ofMillis(1))
        .withTimestampAssigner(new SerializableTimestampAssigner<Order>() {
    @Override
    public long extractTimestamp(Order element, long recordTimestamp) {
        try {
            return DateUtils.strToTimestamp(element.getCreatetime());
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return 0L;
    }
});

orderOperator.assignTimestampsAndWatermarks(orderWatermarkStrategy)
            .keyBy(order -> order.getId())
            .window(TumblingEventTimeWindows.of(Time.seconds(5)))
            .process(new ProcessWindowFunction<Order, Tuple2<Integer, Integer>, Integer, TimeWindow>() {
                @Override
                public void process(Integer key, Context context, Iterable<Order> elements, Collector<Tuple2<Integer, Integer>> out) throws Exception {

                    TimeWindow window = context.window();

                    System.out.println("=========================== window start (key: " + (key) + ")" + window.getStart() + " ===========================");

                    int amount = 0;
                    Iterator<Order> iterator = elements.iterator();
                    while (iterator.hasNext()) {
                        Order order = iterator.next();
                        System.out.println(order);
                        amount += order.getAmount();
                    }

                    System.out.println("=========================== window end (key: " + (key) + ")" + window.getEnd() + " ===========================");

                    out.collect(new Tuple2<>(key, amount));
                }
            }).print("OrderAmount");
```

5秒为一个窗口, 并且是前闭后开的特性, 所以窗口可以分为如下:  
[0...5), [5...10), [10...15), [15...20)...以此类推

我们的测试数据格式如下
```text
1,鞋子,1,2021-06-21 11:01:01
```
假设, 我们传入的数据格式时间如下图展示

![flink水位线事件时间窗口测试-1](https://github.com/basebase/document/blob/master/flink/image/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF%E4%BA%8B%E4%BB%B6%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E6%B5%8B%E8%AF%95-1.png?raw=true)

图中标记了事件时间和对应的watermark时间, 当然watermark真实的数据是一个时间戳而不是一个字符串, 这里只是为了更直观而已。如果能通过这张表理解何时出发计算窗口, 那么对于watermark的基本使用已经掌握了。

下面, 我们可以下执行程序结果
![flink水位线事件时间窗口测试-2](https://github.com/basebase/document/blob/master/flink/image/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF%E4%BA%8B%E4%BB%B6%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E6%B5%8B%E8%AF%95-2.png?raw=true)


我们以11:01:00到11:01:04为一个执行窗口, 但是当我们的事件时间数据不到11:01:08是不会触发11:01:05的window窗口计算的, 假如watermark到了触发的时间戳则相应的window进行计算, 不在等待迟到数据

![flink水位线事件时间窗口测试-3](https://github.com/basebase/document/blob/master/flink/image/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF%E4%BA%8B%E4%BB%B6%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E6%B5%8B%E8%AF%95-3.png?raw=true)
![flink水位线事件时间窗口测试-4](https://github.com/basebase/document/blob/master/flink/image/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF%E4%BA%8B%E4%BB%B6%E6%97%B6%E9%97%B4%E7%AA%97%E5%8F%A3%E6%B5%8B%E8%AF%95-4.png?raw=true)
我们在来看看下面的执行结果, 会发现数据怎么变少了? 这是因为这是两个不同的窗口数据....

