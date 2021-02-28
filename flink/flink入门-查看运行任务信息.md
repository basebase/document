### flink入门-任务运行界面介绍

### 快速上手
上一小节我们已经提交任务, 如何查看正在执行的任务呢?
1. 进入flink首页就能看到运行任务列表
2. 进入Running Job页面查看

![首页查看运行任务](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1%E4%BF%A1%E6%81%AF/%E9%A6%96%E9%A1%B5%E6%9F%A5%E7%9C%8B%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1.png?raw=true)

![running页面查看运行任务](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1%E4%BF%A1%E6%81%AF/running%E9%A1%B5%E9%9D%A2%E6%9F%A5%E7%9C%8B%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1.png?raw=true)


我们可以从页面上看到正在运行的任务, 我们点击进去可以看到任务运行的一个详情
![任务详情页](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1%E4%BF%A1%E6%81%AF/%E4%BB%BB%E5%8A%A1%E8%AF%A6%E6%83%85%E9%A1%B5.png?raw=true)

这个是一个任务详情页面, 可以停止任务并且可以看到任务的一个执行计划, 接收数据/发送数据, 任务并行度及任务数等内容
由于我们发送了5条数据, 所以接收的数据是5字节数是74B, 那么, 数据输出在哪里看呢?

执行任务是在taskmanage上执行的, 所以只要找到对应的taskmanage查看标准输出
![taskmanage页面](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1%E4%BF%A1%E6%81%AF/taskmanage%E9%A1%B5%E9%9D%A2.png?raw=true)
![taskmanage标准输出](https://github.com/basebase/document/blob/master/flink/image/%E8%BF%90%E8%A1%8C%E4%BB%BB%E5%8A%A1%E4%BF%A1%E6%81%AF/taskmanage%E6%A0%87%E5%87%86%E8%BE%93%E5%87%BA.png?raw=true)

