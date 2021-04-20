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

    if (!(upStreamVertex.isSameSlotSharingGroup(downStreamVertex)                   // 2. 上下游节点在同一个slot group中
        && areOperatorsChainable(upStreamVertex, downStreamVertex, streamGraph)     // 3. 上下游算子策略能chain在一起
        && (edge.getPartitioner() instanceof ForwardPartitioner)                    // 4. 上下游节点分区策略是Forward
        && edge.getShuffleMode() != ShuffleMode.BATCH                               // 5. 节点shuffleModel不为BATCH
        && upStreamVertex.getParallelism() == downStreamVertex.getParallelism()     // 6. 上下游节点并行度一致
        && streamGraph.isChainingEnabled())) {                                      // 7. 没有禁用chain

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


static boolean areOperatorsChainable(
        StreamNode upStreamVertex,
        StreamNode downStreamVertex,
        StreamGraph streamGraph) {
    StreamOperatorFactory<?> upStreamOperator = upStreamVertex.getOperatorFactory();
    StreamOperatorFactory<?> downStreamOperator = downStreamVertex.getOperatorFactory();
    if (downStreamOperator == null || upStreamOperator == null) {
        return false;
    }

    // yielding operators cannot be chained to legacy sources
    // unfortunately the information that vertices have been chained is not preserved at this point
    if (downStreamOperator instanceof YieldingOperatorFactory &&
            getHeadOperator(upStreamVertex, streamGraph).isLegacySource()) {
        return false;
    }

    // we use switch/case here to make sure this is exhaustive if ever values are added to the
    // ChainingStrategy enum
    boolean isChainable;

    // 获取上游节点chain类型
    // 如果上游节点是NEVER则不可以chain, 如果是ALWAYS或者HEAD则上游满足条件
    switch (upStreamOperator.getChainingStrategy()) {
        case NEVER:
            isChainable = false;
            break;
        case ALWAYS:
        case HEAD:
        case HEAD_WITH_SOURCES:
            isChainable = true;
            break;
        default:
            throw new RuntimeException("Unknown chaining strategy: " + upStreamOperator.getChainingStrategy());
    }

    // 获取下游节点chain类型
    // 如果下游节点是NEVER或者HEAD则不会被chain到一起, 如果是ALWAYS则可以chain
    switch (downStreamOperator.getChainingStrategy()) {
        case NEVER:
        case HEAD:
            isChainable = false;
            break;
        case ALWAYS:
            // keep the value from upstream
            break;
        case HEAD_WITH_SOURCES:
            // only if upstream is a source
            isChainable &= (upStreamOperator instanceof SourceOperatorFactory);
            break;
        default:
            throw new RuntimeException("Unknown chaining strategy: " + upStreamOperator.getChainingStrategy());
    }

    return isChainable;
}
```


上面的代码约束的条件还挺多的, 总结下来就是:
1. 下游节点只有一个输入边
2. 上下游节点在同一个slot group中
3. 上下游算子策略能chain在一起
    * 3.1: 如果上游节点是NEVER则不可以chain, 如果是ALWAYS或者HEAD则上游满足条件
    * 3.2: 如果下游节点是NEVER或者HEAD则不会被chain到一起, 如果是ALWAYS则可以chain, 对于HEAD_WITH_SOURCES类型则还需要判断上游节点是否为一个Source算子
4. 上下游节点分区策略是Forward
5. 节点shuffleModel不为BATCH
6. 上下游节点并行度一致
7. 没有禁用chain


我们以最开始的wordcount的例子为模板, 当并发度为1时, 执行图如下:

![初始提交任务结构](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90%E9%93%BE/%E5%88%9D%E5%A7%8B%E6%8F%90%E4%BA%A4%E4%BB%BB%E5%8A%A1%E7%BB%93%E6%9E%84.png?raw=true)

我们可以修改flatMap并发数, 单独设置为2, 就会和source算子分解开并且和下游算子也无法链(chain)在一起, 并发度值不一样

![算子并发度不同无法链接](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90%E9%93%BE/%E7%AE%97%E5%AD%90%E5%B9%B6%E5%8F%91%E5%BA%A6%E4%B8%8D%E5%90%8C%E6%97%A0%E6%B3%95%E9%93%BE%E6%8E%A5.png?raw=true)

我们还可以通过在算子(operator)上调用startNewChain断开与上游算子的chain, 并开始一个新的算子链与下游算子chain, 只要满足chain条件

通过在flatMap算子上调用startNewChain, 可以发现与前面的source算子断开chain, 并且我们在sum算子后面也调用startNewChain但不影响和后面的sink算子chain

![调用startNewChain断开上游算子](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90%E9%93%BE/%E8%B0%83%E7%94%A8startNewChain%E6%96%AD%E5%BC%80%E4%B8%8A%E6%B8%B8%E7%AE%97%E5%AD%90.png?raw=true)

但是, 我们将后面的sink并行度修改后就不会再chain在一起了
![验证startNewChain不影响下游算子](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90%E9%93%BE/%E9%AA%8C%E8%AF%81startNewChain%E4%B8%8D%E5%BD%B1%E5%93%8D%E4%B8%8B%E6%B8%B8%E7%AE%97%E5%AD%90.png?raw=true)


我们还可以调用算子的disableChaining不参与chain, 会与前后算子都断开不进行chain, 我么在sum算子上调用disableChaining可以发现其成为一个独立算子任务, 没有在与sink进行chain

![算子调用disableChaining不参与chain](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90%E9%93%BE/%E7%AE%97%E5%AD%90%E8%B0%83%E7%94%A8disableChaining%E4%B8%8D%E5%8F%82%E4%B8%8Echain.png?raw=true)

最后, 我们还可通过调用StreamExecutionEnvironment的disableOperatorChaining方法禁止算子chain

![禁用算子链](https://github.com/basebase/document/blob/master/flink/image/%E7%AE%97%E5%AD%90%E9%93%BE/%E7%A6%81%E7%94%A8%E7%AE%97%E5%AD%90%E9%93%BE.png?raw=true)