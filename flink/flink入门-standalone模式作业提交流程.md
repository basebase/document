### flink入门-standalone模式作业提交流程

本章主要介绍flink在standalone模式下提交作业流程的一个抽象图, 并不是一个详细介绍如何申请资源并运行任务的执行图;

1. 首先启动我们的TaskManager
2. TaskManager向Flink Master(JobManager)的ResourceManager注册资源
3. 用户提交应用程序到Dispatcher组件
4. Dispatcher启动一个JobManager
5. 向ResourceManager申请执行资源
6. 如果资源池不够的话则向TaskManager请求资源
7. JobManager获取到对应资源信息
8. 通过向获取到资源的服务启动任务
9. TaskManager之间进行数据交互

![standalone模式作业提交流程](https://github.com/basebase/document/blob/master/flink/image/standalone%E6%A8%A1%E5%BC%8F%E4%BD%9C%E4%B8%9A%E6%8F%90%E4%BA%A4%E6%B5%81%E7%A8%8B/standalone%E6%A8%A1%E5%BC%8F%E4%BD%9C%E4%B8%9A%E6%8F%90%E4%BA%A4%E6%B5%81%E7%A8%8B.png?raw=true)

### 参考
[深入解读 Flink 资源管理机制](https://www.infoq.cn/article/tnq4vystluqfkqzczesa)  
[Flink运行架构剖析](https://jiamaoxiang.top/2019/10/23/Flink%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84%E5%89%96%E6%9E%90/)  
[FLIP6: 资源调度模型重构](http://www.whitewood.me/2018/06/17/FLIP6-%E8%B5%84%E6%BA%90%E8%B0%83%E5%BA%A6%E6%A8%A1%E5%9E%8B%E9%87%8D%E6%9E%84/)  
[Flink Master 详解](http://matt33.com/2019/12/23/flink-master-5/)

