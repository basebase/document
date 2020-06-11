#### Java并发工具2-线程池

##### 线程池简介

###### 池化技术
在对线程池介绍之前, 我们先来了解一下"池"这个意思, 或许在学习线程池之前, 我们也使用过像"数据库连接池, 内存池"等技术。这些技术都属于"池化技术", 当然也包括了我们的线程池了。

池化技术简单来说: 提前保存大量资源, 以备不时之需。当需要的时候从池中获取, 不需要的时候进行回收, 进行统一管理。

通俗点来说, 当我们去了一个外包公司, 外包公司就是一个大池子招了很多人, 然后外包公司的员工去到各个不同的公司去工作, 等到项目结束后, 回到外包公司继续去下一家公司做项目。


###### 线程池有什么用?
上面对池化技术的一个简单了解, 可以简单的知道线程池可以有效的创建回收并管理我们的线程。那么, 线程池究竟能做些什么呢?

我们首先来看看Oracle文档对线程池的一个简单描述:

**Most of the executor implementations in java.util.concurrent use thread pools, which consist of worker threads. This kind of thread exists separately from the Runnable and Callable tasks it executes and is often used to execute multiple tasks.**

**Using worker threads minimizes the overhead due to thread creation. Thread objects use a significant amount of memory, and in a large-scale application, allocating and deallocating many thread objects creates a significant memory management overhead.**

简单翻译一下:

**我们看到使用线程池可以带来, "减少线程创建所带来的开销"。"分配和取消分配许多线程对象会产生大量内存管理开销"**


举个简单的例子, 假设我们现在要创建100个线程, 如果我们通过原始的Thread类构建可以构建, 但是每个线程都是要消耗内存的, 如果有1w个线程呢? 甚至更多很可能就导致OOM了。但是通过线程池我们无需创建这么多线程, 我们可能创建10个线程, 这个10个线程一直一直处理我们的任务, 直到所有的任务执行完成(如果有点不明白, 请参考池化技术中的比喻)。

[Oracle Java Documentaion pools](https://docs.oracle.com/javase/tutorial/essential/concurrency/pools.html)
