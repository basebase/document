### flink入门-单机版安装部署

#### 快速上手
安装部署flink第一步当然是下载flink, 可以选择最新版本也可以选择比较稳定的旧版本, 我这里选择使用1.11.1
[flink下载地址](https://flink.apache.org/downloads.html#all-stable-releases)


下载并解压flink之后, 我们会看下如下文件  
![flink解压文件](https://github.com/basebase/document/blob/master/flink/image/flink%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2/flink%E8%A7%A3%E5%8E%8B%E6%96%87%E4%BB%B6.png?raw=true)

我们主要关心的是bin, lib以及conf目录;

#### bin目录
bin目录下, 我们只要关心下面红线标注的两个脚本及一个命令
![bin目录结构](https://github.com/basebase/document/blob/master/flink/image/flink%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2/bin%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png?raw=true)

其中一个是启动flink集群一个是停止flink集群脚本, 以及包含一个非常重要的命令 ***flink***  
无论我们是提交任务还是要kill一个任务或者查看正在执行的任务都是需要使用该命令。

#### lib目录
lib目录中存放的是就是flink一些依赖的jar包, 里面包含了flink的table依赖, 日志依赖等其它功能
![lib目录结构](https://github.com/basebase/document/blob/master/flink/image/flink%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2/lib%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png?raw=true)

#### conf目录
该文件夹中包含了许多配置文件, 但是我们主要关心的文件只有下面几个
1. flink-conf.yaml
2. masters
3. workers
4. zoo.cfg

masters文件主要是配置我们JobManager节点, workers文件主要配置工作节点即TaskManager节点, 如果需要使用HA模式则需要配置zoo.cfg文件;

而我们需要配置的则是flink-conf.yaml文件, 这里我们需要了解的几个配置
```yaml
# jobmanager连接地址
jobmanager.rpc.address: localhost

# jobmanager连接端口
jobmanager.rpc.port: 6123

# Flink JobManager进程总内存, 包含了JVM堆内存以及堆外内存
jobmanager.memory.process.size: 1600m

# Flink TaskManager进程总内存, 包含了JVM堆内存以及堆外内存(该节点是工作节点, 所以可以设置比JobManager进程总内存大一些)
taskmanager.memory.process.size: 1728m

# 资源默认一个即一个TaskManager一个资源槽, 可以理解 TaskManager数量 * numberOfTaskSlots
taskmanager.numberOfTaskSlots: 1

# 并行度默认为1
parallelism.default: 1

# flink web页面访问端口
rest.port: 8081
```

其实这些配置我们都不需要修改, 我们即可启动flink服务, 执行start-cluster.sh

![flink-web页面](https://github.com/basebase/document/blob/master/flink/image/flink%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2/flink-web%E9%A1%B5%E9%9D%A2.png?raw=true)