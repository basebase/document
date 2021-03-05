### flink入门-组件介绍

#### 快速上手
当部署并运行flink程序, flink运行有哪些组件在执行并进行交互, 这些组件都是什么角色做什么用的, 本小结将会进行简单的介绍;  
flink采用的也是master/slave主从架构, 一个运行中的flink集群主要由下面两个核心组件构成:
1. JobManager(master)
2. TaskManager(workers)

#### JobManager
JobManager具有协调flink应用程序的功能, 例如何时调度一个task或者一组task, 对完成的task或者失败做出反应, 协调checkpoint等等。
不过该组件是由下面三个不同的组件组成:
1. ResourceManager
2. Dispatcher
3. JobMaster(更多的人可能称为JobManager)

##### ResourceManager
ResourceManager负责flink集群中的资源提供、回收、分配。它管理task slots, 这是flink集群中资源调度的单位;

##### Dispatcher
Dispatcher提供一个REST接口, 用来提交flink应用程序执行, 并为每个提交的作业启动一个新的JobMaster;

##### JobMaster
JobMaster负责管理单个JobGraph执行, flink集群可以同时运行多个任务, 每个作业都有自己的JobMaster;
对于JobMaster(或者称为JobManager)底层源码实现是JobMaster, 该组件(JobManager/JobMaster)的工作范围和集群的JobManager进程工作范围是不同的
JobManager(master)给每个Job创建一个(JobManager/JobMaster)对象, 用(JobManager/JobMaster)来处理这个Job相关协调工作

#### TaskManager
TaskManager(也被称为worker)执行作业task, 并且缓存和交换数据;  
flink集群中必须至少拥有一个TaskManager, 在TaskManager中资源调度的最小单位是task slot。TaskManager中的task slot的数量
表示并发处理的task数量;


#### flink架构
![flink架构](https://raw.githubusercontent.com/basebase/document/d46756923063f0af51e7de969d7ddf8299bd3e65/flink/image/%E7%BB%84%E4%BB%B6%E4%BB%8B%E7%BB%8D/flink%E6%9E%B6%E6%9E%84.svg)

用户提交一个作业, 会将代码转换为JobGraph, 通过Client将生成的JobGraph提交到集群中执行, 此时client可以断开连接(分离模式), 也可以保持连接接收报告(附加模式),
当任务提交到(JobManager也就是我们的JobMaster和我们JobManager进程没有任何关系)就会申请资源, 调度task等工作, TaskManager则可以通过网络进行数据交互;


#### 参考
[Flink架构](https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/concepts/flink-architecture.html)  
[Flink术语](https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/concepts/glossary.html)