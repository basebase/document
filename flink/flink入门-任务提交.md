### flink入门-提交任务

#### 快速上手
上一结我们已经启动flink服务了, 那么如何将我们的任务提交至集群呢?
1. 通过web方式提交
2. 通过flink命令提交

#### 编译项目
既然要提交任务, 我们就需要编译代码, 以最开始编写的wordcount例子作为展示
```shell
mvn clean package 
```

#### web方式提交任务
1. 首先找到Submit New Job进行添加要执行的Jar文件
![web任务提交流程](https://github.com/basebase/document/blob/master/flink/image/%E4%BB%BB%E5%8A%A1%E6%8F%90%E4%BA%A4/web%E4%BB%BB%E5%8A%A1%E6%8F%90%E4%BA%A4%E6%B5%81%E7%A8%8B-1.png?raw=true)

2. 点击上传的Jar文件, 会出现一些输入的参数选项, 执行的类, 接收参数, 并行度等内容, 最后点击提交按钮即可
![web任务提交流程](https://github.com/basebase/document/blob/master/flink/image/%E4%BB%BB%E5%8A%A1%E6%8F%90%E4%BA%A4/web%E4%BB%BB%E5%8A%A1%E6%8F%90%E4%BA%A4%E6%B5%81%E7%A8%8B-2.png?raw=true)

这样, 我们就通过web的形式提交了一个任务

#### flink命令提交
```shell
flink run -c xxx.xxx.x.ClassName jar
```