### flink入门-算子状态

#### 算子状态
算子状态可以用在所有算子上, 算子状态的作用域是算子子任务, 流入这个算子子任务的数据都可以访问和更新这个状态。

具体怎么理解这句话呢?
1. 算子状态不允许跨算子访问, 比如我们的map算子不能访问到reduce算子中的状态, 同理反过来也一样
2. 即使是同一个算子任务, 但是来自不同的子任务也不行, 比如map子任务有4个, 那么这4个子任务状态访问也是独立的, 互不干涉的。

![算子状态共享图解](https://github.com/basebase/document/blob/master/flink/image/flink%E7%8A%B6%E6%80%81/%E7%AE%97%E5%AD%90%E7%8A%B6%E6%80%81%E5%85%B1%E4%BA%AB%E5%9B%BE%E8%A7%A3.png?raw=true)


#### 算子状态实现
状态从本质上来说, 就是flink算子子任务的一种本地数据, 为了保证数据可恢复性, 使用Checkpoint机制来将状态数据持久化输出到存储空间。
状态相关的主要逻辑有两项
1. 将算子子任务本地内存数据在Checkpoint时snapshot写入存储
2. 初始化或重启应用时以一定的逻辑从存储中读出并变为算子子任务的本地内存数据。

Keyed State对这两项内容做了更完善的封装, 开发者可以开箱即用。但是对于Operator State来说, 每个算子子任务管理自己的Operator State,
或者说每个算子子任务上的数据流共享同一个状态, 可以访问和修改状态。

flink的算子子任务上的数据在程序重启、横向伸缩灯场景下不能保证百分百的一致性, 换句话说重启flink应用后, 某个数据流元素不一定会和上次一样, 还能流入该算子子任务上。
因此, 我们需要根据自己的业务场景来设计snapshot和initialize的逻辑。

要实现这两个步骤, 我们需要实现CheckpointedFunction接口类。

```java
public interface CheckpointedFunction {
    // checkpoint时会调用该方法, 我们需要实现具体的snapshot逻辑, 比如把哪些本地状态持久化
	void snapshotState(FunctionSnapshotContext context) throws Exception;
    // 初始化时会调用该方法, 向本地状态中填充数据
	void initializeState(FunctionInitializationContext context) throws Exception;
}
```

在flink的checkpoint机制下, 当一次snapshot触发后, snapshotState方法会被调用, 会将本地状态持久化到存储空间上, 这里我们暂时先不用关心snapshot如何被触发的。
在后续的checkpoint机制会详细介绍。

initializeState在算子子任务初始化时被调用, 初始化包含两种场景
1. 整个flink应用第一次执行, 状态数据被初始化为一个默认值
2. flink作业重启, 之前的状态数据已经持久化到存储空间上, 通过这个方法将存储上的状态数据读出并填充进对应的本地状态中。


目前Operator State主要有三种, 其中ListState和UnionListState在数据结构上都是一种ListState, 还有一种BroadcastState。这里我们主要介绍ListState这种列表形式的状态。这种状态以一个列表的形式序列化存储, 以适应横向扩展时状态重分布的问题。每个算子子任务有零到多个状态S, 组成一个列表ListState[S]。

各个算子子任务将自己状态列表的snapshot到存储, 整个状态逻辑上就可以理解成是将这些列表连接到一起, 组成了一个包含所有状态的大列表。

当作业重启或者横向扩展时, 我们需要将这个包含所有状态的列表重新分布到各个算子子任务上。

ListState和UnionListState的区别在于: ListState是将整个列表状态按照round-robin的模式均匀分布到各个算子子任务上, 每个算子子任务得到的是整个列表的子集。
UnionListState按照广播的模式, 将整个列表发送给每个算子子任务。

```java

public class OperatorStateFunction {

    // flink run -s hdfs://localhost:9000/checkpoint/17a49e53ba97c72c99eea5db061b4024/chk-7/_metadata -p 4 -c com.moyu.flink.examples.state.OperatorStateFunction ./flink-examples-1.0-SNAPSHOT.jar

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

//        env.setStateBackend(new FsStateBackend("hdfs://localhost:50070/checkpoint"));

        // 每秒执行一次checkpoint
        env.enableCheckpointing(10000);

        // 设置模式为精确一次 (这是默认值)
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

        // 确认 checkpoints 之间的时间会进行 500 ms
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);

        // Checkpoint 必须在一分钟内完成，否则就会被抛弃
        env.getCheckpointConfig().setCheckpointTimeout(60000);

        // 同一时间只允许一个 checkpoint 进行
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);

        // 如果我们使用canal取消执行job那么checkpoint会被清除掉, 需要使用RETAIN_ON_CANCELLATION策略, 如果是任务运行失败则无需配置任何策略都可以
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        DataStreamSource<String> socketTextStream =
                env.socketTextStream("localhost", 8888);

        SingleOutputStreamOperator<Integer> result = socketTextStream.filter(str -> !str.equals("") && str.split(",").length == 3)
                .map(str -> {
                    String[] fields = str.split(",");
                    return new Student(Integer.parseInt(fields[0]), fields[1], Double.parseDouble(fields[2]));
                }).returns(Student.class).map(new MyCheckpointedFunctionOperatorStateMap());

        result.print("result");

        env.execute("OperatorStateFunction Test Job");
    }

    private static class MyCheckpointedFunctionOperatorStateMap implements CheckpointedFunction,
            MapFunction<Student, Integer> {

        private Integer count = 0;
        private ListState<Integer> opCntState;

        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
            // 本地变量更新算子状态
            opCntState.clear();
            opCntState.add(count);
        }

        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {
            // 初始化算子状态
            ListStateDescriptor<Integer> listStateDescriptor =
                    new ListStateDescriptor<Integer>("opCnt", Integer.class);
            opCntState = context.getOperatorStateStore().getListState(listStateDescriptor);

            // 通过算子状态初始化本地变量
            Iterator<Integer> iterator = opCntState.get().iterator();
            while (iterator.hasNext()) {
                count += iterator.next();
            }

            System.out.println("init count = " + count);
        }

        @Override
        public Integer map(Student value) throws Exception {
            ++ count;
            return count;
        }
    }
}
```

