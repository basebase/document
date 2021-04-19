### flink入门-算子链

#### 概述
在提交一个flink任务(Job), 会有一些子任务(sub-task)合并在一起执行, 这就是我们要说的算子链, flink会尽可能将算子(operator)的sub-task链接(chain)在一起
形成一个task, 每个task在一个线程中执行。

将算子(operator)链接成task是非常有效的优化:
1. 减少线程之间的切换
2. 减少消息序列化/反序列化
3. 减少数据在缓冲区交换
4. 减少延迟并同时提高吞吐量

#### 实践
那么, 要实现多个算子链在一起是有一定的条件, 我们先来看看下面一段代码

这段代码就是flink实现将算子(operator)链(chain)在一起的代码
```java
// 检测上下游是否能 chain 在一起
public static boolean isChainable(StreamEdge edge, StreamGraph streamGraph) {
    StreamNode downStreamVertex = streamGraph.getTargetVertex(edge);

    // 1. 下游节点只有一个输入边
    return downStreamVertex.getInEdges().size() == 1
            && isChainableInput(edge, streamGraph);
}

private static boolean isChainableInput(StreamEdge edge, StreamGraph streamGraph) {
    StreamNode upStreamVertex = streamGraph.getSourceVertex(edge);
    StreamNode downStreamVertex = streamGraph.getTargetVertex(edge);

    if (!(upStreamVertex.isSameSlotSharingGroup(downStreamVertex)                   // 上下游节点在同一个slot group中
        && areOperatorsChainable(upStreamVertex, downStreamVertex, streamGraph)     // 上下游算子策略能chain在一起
        && (edge.getPartitioner() instanceof ForwardPartitioner)                    // 上下游节点分区策略是Forward
        && edge.getShuffleMode() != ShuffleMode.BATCH                               // 节点shuffleModel不为BATCH
        && upStreamVertex.getParallelism() == downStreamVertex.getParallelism()     // 上下游节点并行度一致
        && streamGraph.isChainingEnabled())) {                                      // 没有禁用chain

        return false;
    }

    // check that we do not have a union operation, because unions currently only work
    // through the network/byte-channel stack.
    // we check that by testing that each "type" (which means input position) is used only once
    for (StreamEdge inEdge : downStreamVertex.getInEdges()) {
        if (inEdge != edge && inEdge.getTypeNumber() == edge.getTypeNumber()) {
            return false;
        }
    }
    return true;
}
```

