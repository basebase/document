### flink入门-运行架构概念介绍

#### 概念介绍
通过前面的组件内容介绍大致了解flink有哪些进程在工作以及如何进行交互, 这是从一个宏观的层面去了解flink架构;
但是, 当我们将一个任务提交flink集群后, 我们看到有一个具体执行图以及下面每个流程下产生的task任务数, 有些任务甚至进行了合并;

下面两张图都是源自同一个程序, 占用的资源数量不一样, 执行任务数不一样, 逻辑执行图都不一样, 这是为什么呢?

![运行架构概念-1.png](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84%E6%A6%82%E5%BF%B5%E4%BB%8B%E7%BB%8D/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84%E6%A6%82%E5%BF%B5-1.png?raw=true)

![运行架构概念-2.png](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84%E6%A6%82%E5%BF%B5%E4%BB%8B%E7%BB%8D/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84%E6%A6%82%E5%BF%B5-2.png?raw=true)


#### flink执行图介绍
flink图很多, 我们只需要先了解flink如何生成这些图以及大致的含义, 大致可分为四层;
StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图

1. StreamGraph: 用户通过Stream API编写的代码生成最初始的图, 用来表示程序的拓扑结构
2. JobGraph: StreamGraph经过优化后生成JobGraph, 提交到JobManager, 主要优化将多个符合条件的节点chain在一起作为一个节点, 从而减少数据在节点之间传输消耗/序列化/反序列化
3. ExecutionGraph: JobManager根据JobGraph生成ExecutionGraph, ExecutionGraph是JobGraph的并行化版本, 是调度层最核心数据结构
4. 物理执行层: JobManager根据ExecutionGraph对Job进行调度后, 在各个TaskManager上部署Task后形成的"图", 并不是一个具体的数据结构

#### flink任务相关
1. Job: 一个Job代表我们提交的一个任务, 可以认为调用执行env.execute()或者executeAsync()就会产生一个Job, 向JobManager提交任务都是以Job为单位
2. parallelism: 并行度, 我们可以通过代码设置某一个算子的并行度, 也可以通过配置文件设置整体的并行度
3. Task: Task是一个逻辑概念, Task会按照并行度分成多个SubTask
4. SubTask: 具体执行调度的基本单元, 每个SubTask都需要一个线程来执行
5. Operator Chains: flink task都是一个一个的算子, 但是flink会尽可能将operator(算子)的subtask链接(chain)在一起, 形成task, 每个task在一个线程中执行

#### flink资源

1. Slot: Flink资源单位, 提供任务执行, 当前Slot只隔离了内存并未隔离CPU, 每个TaskManager根据taskmanager.numberOfTaskSlots配置确定slot个数, 一般根据服务器cpu核心数配置
2. SlotSharingGroup: 为了高效使用计算资源, Flink默认允许同一个Job不同Task的SubTask运行在一个Slot中, 只不过有以下限制条件:
    * 必须是同一个Job
    * 必须是不同Task的SubTask, 为了更好的资源均衡及利用


当把上面这些知识点了解清楚之后, 回过头在上看上面的两张执行图就可以明白
1. parallelism和Slot区别
2. 处理一个任务需要多少个Slot
3. 哪些任务会进行合并