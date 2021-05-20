### flink入门-broadcast-state广播流

#### 实战
Broadcast State是flink支持的一种算子状态(Operator State), 使用Broadcast State可以在flink程序的一个Stream中输入数据记录, 然后将这些数据广播(Broadcast)到下游任务中, 使这些数据能够为被所有Task共享, 比如一些配置数据, 或者我们的维度表实现

想要使用Broadcast State API我们要先创建一个Keyed或者Non-Keyed的DataStream, 然后在创建一个Broadcasted Stream, 最后通过DataStream来连接Broadcast Stream, 这里用的连接就是我们之前学习的connect算子, 这里需要注意的是只能用非广播流数据进行connect, 如果我们的DataStream是一个Keyed Stream在实现process方法就是KeyedBroadcastProcessFunction类型, 如果是一个Non-Keyed Stream哪就是BroadcastProcessFunction类


现在, 我们有一张用户表和一张订单表, 我们将用户表设置为维度表, 订单表为流数据。需要将每个订单数据都关联上为哪个用户方便后面分析

首先创建用户和订单对象

```java
// 订单类
private static class Order {
    private Integer id;         // 订单id
    private Integer goodsId;    // 商品id
    private String goodsName;   // 商品名称
    private String createtime;  // 订单时间
    private String city;        // 订单地址
    private boolean status;     // 订单状态

    public Order() { }

    public Order(Integer id, Integer goodsId, String goodsName, String createtime, String city, boolean status) {
        this.id = id;
        this.goodsId = goodsId;
        this.goodsName = goodsName;
        this.createtime = createtime;
        this.city = city;
        this.status = status;
    }

    // 省去get/set方法
}

// 用户类
private static class User {
    private Integer id;         // 用户id
    private Integer orderId;    // 订单id
    private String name;        // 用户名称
    private Integer sex;         // 用户性别

    public User() { }

    public User(Integer id, Integer orderId, String name, Integer sex) {
        this.id = id;
        this.orderId = orderId;
        this.name = name;
        this.sex = sex;
    }

    // 省去get/set方法
}
```

```java
public class DimBroadcastState {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();


        // 1. 创建两个数据源
        // 1.1 订单数据源, 主流数据
        DataStream<Order> orderStream = env.socketTextStream("localhost", 8888).filter(Objects::nonNull)
                .map(new MapFunction<String, Order>() {
                    @Override
                    public Order map(String value) throws Exception {
                        String[] orderInfos = value.split(",");
                        return new Order(
                                Integer.parseInt(orderInfos[0]),
                                Integer.parseInt(orderInfos[1]),
                                orderInfos[2],
                                orderInfos[3],
                                orderInfos[4],
                                Boolean.parseBoolean(orderInfos[5]));
                    }
                });

        // 1.2 用户数据源, 广播数据
        SingleOutputStreamOperator<User> userStream = env.socketTextStream("localhost", 9999).filter(Objects::nonNull).map(new MapFunction<String, User>() {
            @Override
            public User map(String value) throws Exception {
                String[] users = value.split(",");
                return new User(
                        Integer.parseInt(users[0]),
                        Integer.parseInt(users[1]),
                        users[2],
                        Integer.parseInt(users[3]));
            }
        });


        MapStateDescriptor<Integer, User> userMapStateDescriptor =
                new MapStateDescriptor<Integer, User>("userState", Integer.class, User.class);

        // Keyed Stream实现
        orderStream.keyBy(new KeySelector<Order, Integer>() {
            @Override
            public Integer getKey(Order value) throws Exception {
                return value.getId();
            }
        }).connect(userStream.broadcast(userMapStateDescriptor)).process(new KeyedBroadcastProcessFunction<Integer, Order, User, Tuple2<Order, User>>() {
            @Override
            public void processElement(Order value, ReadOnlyContext ctx, Collector<Tuple2<Order, User>> out) throws Exception {
                ReadOnlyBroadcastState<Integer, User> broadcastState = ctx.getBroadcastState(userMapStateDescriptor);
                User user = broadcastState.get(value.getId());
                out.collect(new Tuple2<>(value, user));
            }

            @Override
            public void processBroadcastElement(User value, Context ctx, Collector<Tuple2<Order, User>> out) throws Exception {
                BroadcastState<Integer, User> broadcastState = ctx.getBroadcastState(userMapStateDescriptor);
                broadcastState.put(value.getOrderId(), value);
            }
        }).print();


        env.execute("DimBroadcast Operator Test Job");
    }
}
```

该例子中会有一个问题, 如果我们数据流先来数广播流是空的, 数据自然无法关联, 不过我看别人例子都是如此, 还需要其他处理方式

#### 参考

[Broadcast State 模式](https://ci.apache.org/projects/flink/flink-docs-master/zh/docs/dev/datastream/fault-tolerance/broadcast_state/)

[Flink使用Broadcast State实现流处理配置实时更新](http://shiyanjun.cn/archives/1857.html)

[Flink实例（五十八）：维表join（二）Flink维表Join实践](https://www.cnblogs.com/qiu-hua/p/13870992.html)

[Flink的流广播(Broadcast State)](https://blog.csdn.net/ZLZ2017/article/details/86411277?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.control)

