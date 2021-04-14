### flink入门-资源分配并行度测试

#### 快速上手
前面已经学习过如何提交任务以及查看任务信息, 假设现在我们修改一下代码

1. 修改算子操作并行度为2
```java
sourceStream.flatMap(new WordCountBatch.WordCountFlatMapFunction())
                .keyBy(0)
                .sum(1).setParallelism(2);
```

2. 修改输出并行度为1
```java
resultStream.print().setParallelism(1);
```

然后我们从新打包编译;

#### 提交任务
当把编译好后的任务再次执行, 但是这次提交我们添加上一个并行度值为3, 提交后我们会发现任务不会被创建而是一直在等待资源中, 如果一直等待不上资源任务就会执行失败。
并且修改后提交的代码执行计划和我们上一节看到的执行计划不一样;

![没有足够资源执行任务](https://github.com/basebase/document/blob/master/flink/image/%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E5%B9%B6%E8%A1%8C%E5%BA%A6%E6%B5%8B%E8%AF%95/%E6%B2%A1%E6%9C%89%E8%B6%B3%E5%A4%9F%E8%B5%84%E6%BA%90%E6%89%A7%E8%A1%8C%E4%BB%BB%E5%8A%A1.png?raw=true)

![没有足够资源执行任务导致失败](https://github.com/basebase/document/blob/master/flink/image/%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E5%B9%B6%E8%A1%8C%E5%BA%A6%E6%B5%8B%E8%AF%95/%E6%B2%A1%E6%9C%89%E8%B6%B3%E5%A4%9F%E8%B5%84%E6%BA%90%E6%89%A7%E8%A1%8C%E4%BB%BB%E5%8A%A1%E5%AF%BC%E8%87%B4%E5%A4%B1%E8%B4%A5.png?raw=true)

***Q: 读取socket的数据源只有一个并行度, 我们不是在提交时指定了默认并行度为什么还是1呢?***  
读取数据源不可能多个任务同时去消费, 只要有一个任务去消费了数据该数据就不存在, 所以该算子操作默认就是一个并行度
不受其它参数影响, 比较特殊;

***Q: Flat Map执行任务数为什么是3?***  
如果我们没有在代码中设置flatMap的并行度则使用我们提交任务时设置的并行度, 所以是3

***Q: Keyed Aggregation任务数为什么是2?***  
根据key分组后的一个聚合操作, 也就是我们的keyby之后的sum操作, keyBy不是一个真正做计算的操作, 可以理解为一个shuffle过程
这个shuffle的并行度是2, 这是我们通过代码设置的一个固定值

对于最后的sink输出, 我想大家都非常清楚为什么是1个任务了

任务在读取并行度优先级为: 算子设置 > 全局env设置 > 命令提交 > 配置文件  
不过有一些点需要注意:
1. 有些算子是无法设置并行度的, 例如: socketTextStream
2. 修改并行度会影响任务分配, 进而改变task数量, 如果Slot数量不满足要求会导致任务无法拥有足够资源运行从而导致失败(例如本例)


那么怎么解决当前的问题呢?
1. 修改flink的Slot数量
2. 修改任务的并行度数量

这里我们修改flink Slot配置, 并重启flink集群

```yaml
# 设置Slot为4即可满足当前任务所需要
taskmanager.numberOfTaskSlots: 4
```

![更新资源数量](https://github.com/basebase/document/blob/master/flink/image/%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E5%B9%B6%E8%A1%8C%E5%BA%A6%E6%B5%8B%E8%AF%95/%E6%9B%B4%E6%96%B0%E8%B5%84%E6%BA%90%E6%95%B0%E9%87%8F.png?raw=true)

当我们再次提交任务后, 可以正常运行, 但是我们发现我们还有一个Slot没有被使用, 这又是为什么呢? 
![正常启动任务](https://github.com/basebase/document/blob/master/flink/image/%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E5%B9%B6%E8%A1%8C%E5%BA%A6%E6%B5%8B%E8%AF%95/%E6%AD%A3%E5%B8%B8%E5%90%AF%E5%8A%A8%E4%BB%BB%E5%8A%A1.png?raw=true)
![还剩下的资源数](https://github.com/basebase/document/blob/master/flink/image/%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E5%B9%B6%E8%A1%8C%E5%BA%A6%E6%B5%8B%E8%AF%95/%E8%BF%98%E5%89%A9%E4%B8%8B%E7%9A%84%E8%B5%84%E6%BA%90%E6%95%B0.png?raw=true)

如果我们现在在提交一次任务, 这次提交的任务能否正常执行? 答案是不能正常执行, 会出现最初的问题没有资源可以使用;