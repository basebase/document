### flink入门-运行架构slot与parallelism区别

#### 概述
通常我们总会把parallelism与slot进行比较, 确实入门学习的时候不是很理解这两个参数具体作用, 我们最初看到parallelism和slot的概念是通过配置文件, 两个参数默认值都是设置为1;
但是对于parallelism我们还可以通过哪些方式进行设置呢?

##### 如何设置并行度
1. 通过env.setParallelism()设置一个并行度
2. 给每个算子设置并行度, 例如我们的flatMap/sum/sink等
3. 通过客户端提交-p参数设置并行度
4. 根据flink配置文件设置并行度

以上参数配置执行顺序为: 算子级别 > env级别 > 客户端 > 配置文件

对于parallelism(并行度)的设置可以是动态的, 并且可以单独配置某个算子可以覆盖默认配置的并行度

##### slot
但是, 对于slot的配置方式呢? 除了通过配置文件配置并无法通过代码进行配置, 属于静态配置资源, 一旦配置后无法通过其他方式进行更改;

TaskManager是运行在不同节点上的JVM进程(process), 该进程会拥有一定的资源, 如: 内存, cpu, 网络等;  
为了控制一个TaskManager能接受多少个task, flink提出slot的概念;

flink中的计算资源通过slot来定义, 每个slot代表TaskManager的一个固定大小的资源子集, 例如: 一个拥有3个slot的TaskManager, 会将其管理的内存平均分成三等分给各个slot。
将资源slot化意味着来自不同job的task不会为了内存而竞争, 因为每个slot都拥有一定数量的内存储备。需要注意的是, ***slot目前仅隔离内存, 并没有进行cpu隔离***

每个TaskManager都有一个slot, 也就意味着每个task运行在独立的JVM中, 如果每个TaskManager有多个slot, 则多个task运行在同一个JVM中。而在同一个JVM进程中的task
可以共享TCP连接和心跳信息, 可以减少数据的网络传输, 也能共享一些数据结构, 一定程度上减少了每个task消耗;


对比发现, parallelism是可以细化到算子级别的, 可以动态设置每个算子并行度从而更加高效处理数据

通过上面两点, 可以大概了解parallelism与slot的区别
1. slot属于静态资源, 一旦设置具体值就保持不变, 想要更新则需要修改配置文件并且重启集群服务; parallelism属于动态资源, 可以通过程序动态为每个算子设置不同并行度值
2. slot代表每个TaskManager最大的并发能力, parallelism代表实际需要的并发数量

#### 资源使用例子


