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


**Thread vs ThreadPool**

举个简单的例子, 假设我们现在要创建100个线程, 如果我们通过原始的Thread类构建可以构建, 但是每个线程都是要消耗内存的, 如果有1w个线程呢? 甚至更多很可能就导致OOM了。并且如此之多的线程需要进行上下文的切换也是极其消耗时间的。但是通过线程池我们无需创建这么多线程, 我们可能创建10个线程, 这个10个线程一直一直处理我们的任务,直到所有的任务执行完成。 在此过程中执行相同等级的任务, 使用线程池极大的减少了上下文的切换所花费的时间。

在比如说, 我们现在有1w个线程, 如果要全部终止这些线程呢? 我们只能遍历出每个线程进行中断操作了! 但是使用线程池相关API就可以很方便的关闭线程池并终止线程任务。

可以看到如果我们使用Thread来创建线程会带来下面一些问题:
  1. 创建N多线程导致难以维护
  2. 线程的上下文切换
  3. 创建过多的线程量可能导致OOM

而线程池就很好的解决了上述的问题。


**既然说了线程池那么多的好处, 那线程池在存在哪些缺点呢?**

至于缺点每个人可能看点不同, 我更想称为一些注意点, 不过stackoverflow上有人对线程池的缺点讨论过, 我觉得@Solomon Slow用户说的挺对的。
[Are there any disadvantages of using a thread pool?](https://stackoverflow.com/questions/22663194/are-there-any-disadvantages-of-using-a-thread-pool)

使用线程池, 应该要注意下面几点:
  * 使用过大的线程池包含太多线程, 这会影响应用程序的性能; 如果线程池太小可能不会带来期望的性能提升。

  * 避免阻塞线程太长时间, 可以指定最大等待时间, 再此之后等待任务被拒绝或重新添加到队列中。

  * 死锁问题

对于死锁问题, 我也看了一些博客下面有人评论说这不是线程池需要注意的问题, 是线程本身需要注意的问题, 其实说到底线程池只是帮助我们管理维护线程, 其本质问题都是线程问题。

**既然使用线程池如此的好, 哪我们为什么还要单独创建线程呢? 都直接使用线程池不是更好?**

这个主要看场景了, 对于有很多需要不断处理的逻辑任务, 并希望并发执行可以使用线程池。但是对于一些临时启动一个线程任务或一些IO相关任务可以创建自己的线程。

[什么时候使用线程池, 基于c#?](https://stackoverflow.com/questions/145304/when-to-use-thread-pool-in-c)


[Oracle Java Documentaion pools](https://docs.oracle.com/javase/tutorial/essential/concurrency/pools.html)

[thread-pool-vs-many-individual-threads](https://stackoverflow.com/questions/11700763/thread-pool-vs-many-individual-threads)

[Getting the Most Out of the Java Thread Pool](https://dzone.com/articles/getting-the-most-out-of-the-java-thread-pool)


##### 线程池创建

在我们创建线程池之前, 我们先来看看线程池中的参数。如果不了解线程池中参数机制, 当我们遇到下面的问题, 可以会有点懵逼哦。

**1. 现有一个线程池, 参数corePoolSize为5, maximnumPoolSize为10, BlockingQueue阻塞队列长度为5, 此时有4个任务同时进来, 线程池会创建几条线程?**

**2. 如果4个任务还没处理完, 这时又同时进来2个任务, 线程池又会创建几条线程还是不会创建?**

**3. 如果前面6个任务还没处理完, 这时又同时进来5个任务, 线程池又会创建几条线程还是不会创建?**


如果, 上述的问题觉得没问题的话, 参数介绍就可以直接跨过去了, 如果有点迷糊, 我们就往下学习来解开这层迷雾。


###### 线程池参数介绍

我们先将线程池对象的构造方法贴出来, 来看看具体究竟有多少个参数。

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

线程池所需要的参数值有7个, 下面就一一介绍每个参数的含义

**corePoolSize**  
线程池核心线程数, 核心线程不会被回收, 即使没有任务执行, 也会保持空闲状态。如果线程池中的线程**小于**核心线程数, 则在执行任务时创建。

**workQueue**  
当线程超过核心线程数之后, 新的任务就会处在等待状态, 并存在于workQueue中

**maximumPoolSize**  
线程池允许最大线程数, 当线程数达到corePoolSize并且workQueue队列也满了之后, 就继续创建线程直到maximumPoolSize数量

**handler**  
当线程数超过corePoolSize, 且workQueue阻塞队列已满, maximumPoolSize线程也已经超过之后, 执行拒绝策略。
