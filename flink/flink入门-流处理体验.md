### flink入门-流处理体验

#### 快速上手
上一节已经学习过批处理程序, 对于流处理其实变动的代码并不算多, 但需要注意的有下面几点:
1. 创建的执行环境不同
2. 输出内容
3. 任务真正执行

#### 创建运行环境
对于运行环境来说, 批处理的时候我们使用的是ExecutionEnvironment来获取的上下文环境, 但这仅限批处理时候使用, 
要获取流处理的上下文环境只需要在ExecutionEnvironment前面加上Stream即可, 如下:
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```
这就可以创建我们流处理需要的执行环境, 非常的简单, 只要在前面加入Stream就可以了。


#### 读取数据源
对于流处理来说也可以读取文件数据, 我们先来看看读取文件内容会发生什么。

读取文件数据
```java
DataStream<String> sourceStream = env.readTextFile(inPath);
```
这里需要注意的是, 当调用readTextFile()方法时返回的是DataStreamSource类型, 但是深入该类后发现其底层是DataStream类
所以flink流处理API被称为DataStream API

批处理使用DataSet API, 批处理和流处理都可以使用DataStream API, 一般DataSet处理有限的数据集而DataStream一般处理无限的数据集
这里有限的数据集指的是这个数据不会再增长了, 例如我们的文件固定就是N条, 而无限的数据集则不知道有多少条如我们的消息队列一直生产数据。


#### 数据统计
对于数据切分, 我们可以使用批处理时创建的flatMap类因为流和批处理的FlatMapFunction是同一个类, 无需在实现一个新的处理类。
但是, 我们要注意的是流处理中不存在groupBy方法, 而是使用keyBy方法, 当然也是可以传入位置值的。

```java
DataStream<Tuple2<String, Integer>> resultStream = sourceStream.flatMap(new WordCountBatch.WordCountFlatMapFunction())
                .keyBy(0)
                .sum(1);
resultStream.print();
```

现在, 如果我们执行任务, 会出现什么情况? 答案是没有任何输出, 我们的任务没有任何执行。

为什么批处理的时候可以正确的把内容输出而流处理就不行呢, 我们先看看DataSet和DataStream的print方法源码

DataSet.print()方法
![dataset-print](https://github.com/basebase/document/blob/master/flink/image/%E6%B5%81%E5%A4%84%E7%90%86%E4%BD%93%E9%AA%8C/dataset-print.png?raw=true)

DataStream.print()方法
![datastream-print](https://github.com/basebase/document/blob/master/flink/image/%E6%B5%81%E5%A4%84%E7%90%86%E4%BD%93%E9%AA%8C/datastream-print.png?raw=true)

对于流来说仅仅只是添加了一个sink而已, 对于sink我们后续会介绍。而我们上面编写的程序可以分为下面几个基本部分
1. 获取execution environment(执行环境)
2. 加载/创建初始数据
3. 数据的转换(切分数据为二元组)
4. 指定将计算结果放在何处(也是我们的sink, 这里我们仅仅是输出Std Out)
5. 触发程序执行

没错, 现在我们要做的是就是第五步触发程序, 也就是调用执行环境的execute()方法, 才会正式启动并运行我们的程序;

#### 触发执行程序
```java
env.execute();
```

只有在调用此方法后, 程序才可以被触发执行。


#### 内容输出
![流处理-输出-1](https://github.com/basebase/document/blob/master/flink/image/%E6%B5%81%E5%A4%84%E7%90%86%E4%BD%93%E9%AA%8C/%E6%B5%81%E5%A4%84%E7%90%86-%E8%BE%93%E5%87%BA-1.png?raw=true)

这里有三个问题我们要注意:
1. 为什么会出现重复的key值数据, 而不是最终数据?
2. 输出前面的数字>是什么意思?
3. 流处理为什么程序会停止?

第一个问题, 正因为是流处理, 我们来一条数据就处理一条数据, 当第一条hello被处理时我们记录为1, 当第二条hello被处理我们可以发现其状态变更为2, 这里我们只要知道flink的状态管理(Working with State) 

第二个问题, 这是flink并行执行数(编号), 和我们机器的cpu数量有关, 比如你有8核可能输出编号就会是1~8, 当然我们也可以通过代码进行设置并行度  
第三个问题, 其实很好理解, 文件数据是一个有界的数据, 当读取完后程序就执行完成, 但是真正的流程序是一直执行的, 下面也会利用socket来实现一个正经的流处理程序  


#### 读取socket数据
通过nc命令启动一个服务使用flink连接并读取写入的数据
```java
DataStream<String> sourceStream = env.socketTextStream("localhost", 8888, "\n");
```
并且, 我们还可以设置并行度为1;

```java
env.setParallelism(1);
```

![流处理-输出-2](https://github.com/basebase/document/blob/master/flink/image/%E6%B5%81%E5%A4%84%E7%90%86%E4%BD%93%E9%AA%8C/%E6%B5%81%E5%A4%84%E7%90%86-%E8%BE%93%E5%87%BA-2.png?raw=true)

可以看到的是我们的程序是一直在执行的状态, 并且并行度设置为1后前面就不在出现数字>的内容了。