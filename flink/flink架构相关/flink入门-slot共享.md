### flink入门-slot共享

#### 概述
当提交运行一个任务, 我们会发现source和flatMap是在同一个slot中或者是keyBy/Window在同一个slot中去执行, 又或者其它几个算子占据一个slot, 这就是slot共享

#### flink slot sharing(slot共享)
默认情况下, flink允许sub-task共享slot, ***条件是它们来自同一个Job不同Task的sub-task***, 结果可能一个slot持有该job的整个pipline。

共享slot有两个好处:
* flink集群所需的task slot数与job中最大的并行度一致。***也就是说我们不需要在去计算一个程序总共会启多少个task了。***
* 更充分的资源利用, 如果没有slot共享, 那么非密集型sub-task(source/flatmap)就会占用同密集型sub-task(keyAggregation/sink/window)一样多的资源。简单理解为: map算子计算并不复杂有很多空闲资源可以被使用, 如果没有slot共享就是一种浪费, 如果使用slot共享则可以把密集型任务放在同一个slot中处理, 不至于浪费空闲资源。


SlotSharingGroup是flink用来实现slot共享类, 它尽可能的让sub-task共享一个slot。相应的还有一个CoLocationGroup类用来强制将sub-task放到同一个slot中;

如何判断一个算子(operator)属于哪个slot共享组, 默认情况下所有算子(operator)都属于 ***default***共享组, 也就是说默认情况下所有算子(operator)都可以共享一个slot。

共享组是继承式的, 如果上一个共享组是default下一个算子则也是default, 如果上一个算子共享组是"AAA"则下一个算子共享组名称也是AAA。我们可以通过调用slotSharingGroup方法来设置算子的共享组名称

通过调用slotSharingGroup方法强制指定算子(operator)共享组可以有效防止不合理的共享, 当一个slot中包含整个任务pipline, 所有sub-task都在一台机器上并使用一个slot内存, 线程切换的开销等问题都是不合理的并影响程序性能, 这个时候为其它算子设置不同共享组, 通过其它slot可以有效的平摊计算能力;

这里有个问题需要注意: 当出现两个共享组[defautl, AAA], 如果共享组default算子最大并行度为4, 共享组AAA算子最大并行度为2则需要多少个slot? 

答案是6个slot, 由于不是同一个共享组我们需要取出不同共享组最大算子并行度累加即: default中的4加上AAA中的2。

上面的文字看起来太枯燥了, 我们透过下面两张图更加直观的去理解

1. 整个任务(Job)的pipline在一个slot中执行, 我们用最开始的wordcount进行举例, 使用默认的并行度1, 在flink中执行会入下图:

![一个slot包含整个pipline](https://github.com/basebase/document/blob/master/flink/image/slot%E5%85%B1%E4%BA%AB%E7%BB%84/%E4%B8%80%E4%B8%AAslot%E5%8C%85%E5%90%AB%E6%95%B4%E4%B8%AApipline.png?raw=true)

所有的任务都在一个slot中执行, 大大的减少了数据传输等成本, 并且只占用了一个slot

2. 为不同算子设置不同的slot共享组, 占用多个slot执行任务, 我们将wordcount例子中flatMap算子设置共享组"AAA", 这样flatMap及其下游算子slot共享组名称都为"AAA", 并且设置sink算子共享组为"ACC"

```java
DataStream<Tuple2<String, Integer>> resultStream = sourceStream.flatMap(new WordCountBatch.WordCountFlatMapFunction()).slotSharingGroup("AAA").setParallelism(2).keyBy(0).sum(1);
resultStream.print().slotSharingGroup("ACC");
```

![不同slot组占用slot量](https://github.com/basebase/document/blob/master/flink/image/slot%E5%85%B1%E4%BA%AB%E7%BB%84/%E4%B8%8D%E5%90%8Cslot%E7%BB%84%E5%8D%A0%E7%94%A8slot%E9%87%8F.png?raw=true)

取出每个共享组中最大算子并行度值累加, 我们需要4个slot才能运行任务