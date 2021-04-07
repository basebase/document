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

每个TaskManager都有一个slot(一个slot内执行一个task, 所以task之间是相互隔离的), 如果TaskManager只有一个slot, 则运行在该slot的task将独享JVM, 如果每个TaskManager有多个slot, 则多个task运行在同一个JVM中。而在同一个JVM进程中的task
可以共享TCP连接和心跳信息, 可以减少数据的网络传输, 也能共享一些数据结构, 一定程度上减少了每个task消耗;


#### 资源使用例子

![flink任务资源图](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB/flink%E4%BB%BB%E5%8A%A1%E8%B5%84%E6%BA%90%E5%9B%BE.jpeg?raw=true)

上图中, 我们有任务A,B,C,D,E并且拥有两个两个TaskManager每个TaskManager配置slot值为2, 但是运行时发现不同子任务运行在同一个slot中(后面介绍多个task如何共享一个slot资源), 那么该任务(Job)运行需要多少个slot?

其实要判断一个任务(Job)需要多少slot非常容易, 我们只需要看 ***该任务(Job)设置算子最大并行度就是我们需要的slot***

其中任务A、B、D并行度都为4, 任务C、E并行度都为2, 总共子任务为: 4 + 4 + 4 + 2 + 2 = 16  
但是所需的slot则只要4个即可运行;

#### slot与parallelism区别

下图中我们有三个TaskManager并且每个TaskManager都设置了3个slot所以总共有9个task slot可以使用, slot代表TaskManager并发执行能力
![slot与parallelism区别-1](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB/slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB-1.jpeg?raw=true)

第一个例子, 我们程序使用flink配置文件中默认的并行度1, 则使用一个slot其余8个slot都是空闲的
![slot与parallelism区别-2](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB/slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB-2.jpeg?raw=true)

第二个例子通过客户端提交或者通过env设置并行度2, 则使用2个slot剩余7个空闲slot, 第三个例子设置并行度为9则将所有slot都占用没有空闲资源
![slot与parallelism区别-3](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB/slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB-3.jpeg?raw=true)

第四个例子中, 我们其余任务并行度都是9但是sink我们设置的并行度为1(如果不设置sink的并行度在写入文件由于是多个线程写入会出现数据乱序等问题, 所以这里设置1是合适的)
![slot与parallelism区别-4](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB/slot%E4%B8%8Eparallelism%E5%8C%BA%E5%88%AB-4.jpeg?raw=true)

通过上面了解, 对于parallelism与slot的区别总结如下
1. slot属于静态资源, 一旦设置具体值就保持不变, 想要更新则需要修改配置文件并且重启集群服务; parallelism属于动态资源, 可以通过程序动态为每个算子设置不同并行度值
2. slot代表每个TaskManager最大的并发能力, parallelism代表一个任务(Job)实际需要的并发数量, 如果一个任务设置的并发值大于slot值则会出现资源不足的情况任务超时报错