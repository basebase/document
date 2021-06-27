### flink入门-状态概念

#### 状态管理概念介绍
流式应用分为有状态和无状态两种, 不过大部分流式应用都是有状态的, 其实要理解有状态和无状态应用非常简单, 无状态应用就是处理数据不依赖任何之前存储的状态信息, 有有状态的应用则需要根据之前计算结果进行累加或者其它操作

##### 生活中的例子
小明经常去一家饭店吃饭, 但在没有办理这家餐厅的会员卡, 小明每一次去店员都是按照餐品实际价格计算的菜单金额, 店家不会因为小明每次去他们家吃饭从而打折, 就和陌生人对待一样, 这就是一个无状态的算子, 店员记不住任何事情。

但是小明决定不在继续做冤大头了, 当即在店家那里办理了一张会员卡, 这样小明在下一次去消费的时候拿出会员卡店员在查询消费金额后给出对应的打折优惠, 然后在更新消费金额以便下次优惠粒度更大。这就是一个有状态的算子, 依赖之前的数据记录。


##### 流应用中的状态
状态在数据处理中无处不在, 任何一个稍微复杂的计算都要使用。为了生成结果, 函数会在一段时间或基于一定个数的事件来累计状态, 下面就是一些使用状态的场景

1. 数据流中的重复数据, 我们要对重复数据去重, 需要记录哪些数据已经流入过应用, 当新数据流入时, 根据已流入过的数据判断去重。
2. 检查输入流是否符合某个特定的模式, 需要将之前输入流的元素以状态的形式缓存下来, 比如一些注册渠道存在恶意刷单行为。
3. 对一个时间窗口内的数据进行聚合分析, 分析时间窗口内某些指标是否超过设定的阈值进行报警等。

我们通过下图来看看有状态和无状态应用之间的区别, 输入记录由黑条表示
![无状态流处理与有状态流处理的区别](https://github.com/basebase/document/blob/master/flink/image/flink%E7%8A%B6%E6%80%81/%E6%97%A0%E7%8A%B6%E6%80%81%E6%B5%81%E5%A4%84%E7%90%86%E4%B8%8E%E6%9C%89%E7%8A%B6%E6%80%81%E6%B5%81%E5%A4%84%E7%90%86%E7%9A%84%E5%8C%BA%E5%88%AB.png?raw=true)

无状态流处理每次只转换一条输入记录, 并且仅根据最新的输入记录输出结果(白条)。有状态流处理维护所有已处理记录的状态值, 并根据每条新输入的记录更新状态, 输出记录(灰条)反映的是综合考虑多个事件之后的结果。


flink一个算子有多个子任务, 每个子任务分布在不同实例上, 我们可以把状态理解为某个算子子任务业务逻辑所需要访问的本地或者实例变量。

下图展示了某个任务和它状态之间的典型交互过程。
![带有状态的流处理任务](https://github.com/basebase/document/blob/master/flink/image/flink%E7%8A%B6%E6%80%81/%E5%B8%A6%E6%9C%89%E7%8A%B6%E6%80%81%E7%9A%84%E6%B5%81%E5%A4%84%E7%90%86%E4%BB%BB%E5%8A%A1.png?raw=true)

任务首先会接收一些输入数据, 在处理这些数据过程中, 任务对其状态进行读取或更新, 并根据状态和输入数据计算结果。

当任务收到一个新的记录后, 首先会访问状态获取当前统计的记录数目, 然后把数据增加并更新状态, 最后将更新后的数目发送出去。

应用读写状态的逻辑非常简单, 但难点在于如何高效、可靠的管理状态。这其中就包括如何处理数量巨大、可能超出内存的状态, 如何保证发生故障时状态不会丢失。所有和状态一致性、故障处理以及高效存取相关的问题都由flink来搞定, 开发人员只需要专注于自己的业务逻辑。

##### Flink几种状态类型
flink有两种基本类型的状态: 托管状态(Managed State)和原生状态(Raw State)

![Managed State & Raw State](https://github.com/basebase/document/blob/master/flink/image/flink%E7%8A%B6%E6%80%81/ManagedState&RawState.png?raw=true)

两者区别如下:
* 从状态管理方式来说, Managed State由Flink Runtime管理, 自动存储, 自动恢复, 在内存管理上有优化; 而Raw State需要用户自己管理, 需要自己序列化, Flink不知道State中存入的数据是什么结构, 只有用户自己知道, 需要最终序列化为可存储的数据结构。

* 从状态数据结构来说, Managed State支持已知的数据结构, 如Value、List、Map等。而Raw State只支持字节数组, 所有状态都需要转为二进制字节数组才可以。

* 从推荐使用场景来说, Managed State大多数情况下均可使用, 而Raw State是当Managed State不够用时, 比如需要自定义Operator时, 推荐使用Raw State。


###### Keyd State & Operator State
Managed State分为两种, 一种是Keyd State; 另外一种是Operator State。在Flink Stream模型中, DataStream经过keyBy后可转变为KeyedStream。

每个Key对应一个State, 即一个Operator实例处理多个Key, 访问相应多个State, 并由此衍生了Keyed State。Keyed State只能用在KeydStream的算子中, 即在整个程序中没有keyBy的过程就没有办法使用KeydStream。

相比而言, Operator State可以用于所有算子, 相对于数据源有一个更好的匹配方式, 常用于Source, 例如FlinkKafkaConsumer。相比Keyed State, 一个Operator实例对应一个State, 随着并发的改变, Keyed State中, State随着Key在实例间迁移, 比如原来有1个并发, 对应的API请求过来, /api/a和/api/b都存放在这个实例中, 如果请求量变大, 需要扩容, 就会把/api/a的状态和/api/b的状态分别放在不同的节点。由于Operator State没有key, 并发改变时需要选择状态如何重新分配。

其中内置了2种分配方式: 一种是均匀分配, 另外一种是将所有的State合并为全量State在分发给每个实例。

在访问上, Keyd State通过RuntimeContext访问, 这需要Operator是一个Rich Function。Operator State需要自己实现CheckpointedFunction或ListCheckpointed接口。

在数据结构上, Keyd State支持的数据结构有: ValueState、ListState、ReducingState、AggregatingState和MapState；
而Operator State支持的数据结构相对较少, 只有ListState

![KeyedState&OperatorState](https://github.com/basebase/document/blob/master/flink/image/flink%E7%8A%B6%E6%80%81/KeyedState&OperatorState.png?raw=true)