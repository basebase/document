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
线程池核心线程数, 核心线程不会被回收, 即使没有任务执行, 也会保持空闲状态。线程池在完成初始化后, 默认情况下线程池中没有任何线程, 如果线程池中的线程**小于**核心线程数, 则在执行任务时创建。

**workQueue**  
当线程超过核心线程数之后, 新的任务就会处在等待状态, 并存在于workQueue中。
常用workQueue如下:
  1. SynchronousQueue(直接交接): 这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，如果所有线程都在工作怎么办(使用它一般将maximumPoolSize设置为Integer.MAX_VALUE，即无限大)
  2. LinkedBlockingQueue(这个无界队列, 如果处理速度赶不上生产速度可能会引发OOM)
  3. ArrayBlockingQueue(有界队列)


**maximumPoolSize**  
线程池允许最大线程数, 当线程数达到corePoolSize并且workQueue队列也满了之后, 就继续创建线程直到maximumPoolSize数量

**handler**  
当线程数超过corePoolSize, 且workQueue阻塞队列已满, maximumPoolSize线程也已经超过之后, 执行拒绝策略。

**keepAliveTime**  
如果线程池当前线程超过corePoolSize, 那么多余的线程空闲时间超过keepAliveTime, 就会被终止。

**unit**  
keepAliveTime的时间单位  


**ThreadFactory**  
创建线程的工厂, 默认使用Executors.defaultThreadFactory(), 创建出来的线程都在同一个线程组, 同样的优先级并且都不是守护线程。当然也可以自己指定ThreadFactory用以改变线程名, 线程组, 优先级等。


在了解上面4个参数之后, 我们可以整理出线程池创建线程的规则如下:
  1. 如果线程数小于corePoolSize, 即使其他工作线程处于空闲状态, 也会**创建**一个新的线程来运行任务。

  2. 如果线程数大于或者等于corePoolSize但少于maximumPoolSize, 则将任务放入队列中

  3. 如果队列也满了, 并且线程数小于maximumPoolSize, 则创建一个新线程来运行任务。


  4. 如果队列满了, 并且线程数大于等于maximumPoolSize, 则拒绝该任务。

对于上述流程, 整理如下一张流程图:

![线程池提交任务策略](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%8F%90%E4%BA%A4%E4%BB%BB%E5%8A%A1%E7%AD%96%E7%95%A5.png?raw=true)

从上图中可以知道线程池创建线程规则如下:
  1. corePoolSize
  2. workQueue
  3. maximumPoolSize
  4. handler


有了上述的知识, 我们在回到刚开始提出的问题, 是不是就很清楚了。
1. 问题1: 4个任务同时进来, 此时会创建4个线程。
2. 问题2: 如果4个任务没处理完, 新加入2个任务, 会在创建一个线程, 另外一个任务会加入到队列中
3. 问题3: 如果上述的6个任务都没处理完, 在加入了5个, 把这5个任务加入队列中, 那么此时队列已满, 会使用maximumPoolSize参数在创建一个临时线程处理任务。


最后, 我们整理一下线程增减的特点:
  1. corePoolSize和maximumPoolSize如果相同, 就会创建一个固定大小的线程池(就算队列满了, 也不会在创建线程了。)

  2. 线程池希望保持较少的线程数, 并且只有在负载变得很大时才增加它。

  3. 将maximumPoolSize设置为很高的值, 例如Integer.MAX_VALUE, 就可以允许线程池容纳任意数量的并发任务(假设我们的队列数量是100, 队列满了之后, maximumPoolSize就会创建临时线程处理, 但是由于Integer.MAX_VALUE基本不会饱和, 可能会创建1k-2k甚至更多的临时线程去处理。)

  4. 只有在队列满了之后才会创建多于corePoolSize的线程, 所以如果使用无界队列(例如: LinkedBlockingQueue), 那么线程数就不会超过corePoolSize。(线程池创建线程的规则就是当队列满了之后才创建临时线程, 现在我们的队列永远都不会满所以线程数永远都是核心线程数, 即使设置了maximumPoolSize也是无效的, 这个和我们上面的第3点不一样, 第三点是创建出N多个线程, 而这个无法创建新的线程。)


[线程池，这一篇或许就够了](https://liuzho.github.io/2017/04/17/%E7%BA%BF%E7%A8%8B%E6%B1%A0%EF%BC%8C%E8%BF%99%E4%B8%80%E7%AF%87%E6%88%96%E8%AE%B8%E5%B0%B1%E5%A4%9F%E4%BA%86/)

[你都理解创建线程池的参数吗？](http://objcoding.com/2019/04/11/threadpool-initial-parameters/)

[ThreadPoolExecutor — Its behavior with Parameter](https://medium.com/@ashwani.the.tiwari/threadpoolexecutor-its-behavior-with-parameter-5e2979381b65)



###### 线程池创建
线程池的创建有两种方式, 一种是手动创建, 另外一种是自动创建。
这里的自动创建指的是使用JDK封装好的构造函数, 而手动创建则是我们自己写入上述对应的参数值。

使用手动创建会比自动创建更优, 因为这样可以让我们更加明确线程池运行规则, 避免资源耗尽的风险。


现在说手动创建比自动创建要好, 那么自动创建为什么不好呢? 我们先来看看JDK自带的一些创建方法, 看看其中有哪些弊端。

**newFixedThreadPool(固定线程池创建)**  
我们首先来看看newFixedThreadPool这个方法如何创建一个线程池。

```java

/**
 *      描述:     演示newFixedThreadPool的使用
 */
public class FixedThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService executorService =
                Executors.newFixedThreadPool(5);
        for (int i = 0; i < 100; i++) {
            executorService.execute(task());
        }
    }

    private static Runnable task() {
        return () -> {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName());
        };
    }
}
```

我们使用Executors.newFixedThreadPool创建出只有5个线程的线程池。而且输出的线程名字都是1~5之间的, 没有多出来的临时线程?

我们点击进入newFixedThreadPool()方法看看,

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
可以看到newFixedThreadPool传入的corePoolSize和maximumPoolSize是一样大小的, 所有肯定不会有新的线程被创建出来, 并且可以看到keepAliveTime线程活跃时间设置为0秒, 这个是没问题的毕竟都不会新建临时线程, 更本就不需要活跃时间进行销毁临时线程数。

还有一个点就是, newFixedThreadPool()方法传入的workQueue参数是LinkedBlockingQueue(无界队列)这也就意味着, 我们的队列永远不会满, maximumPoolSize也永远不会起作用, 这就会导致刚才我们说的一个问题如果提交任务比消费任务快很多, 任务队列很有可能就会出现OOM。

我们通过下面一个例子展示使用newFixedThreadPool出现OOM错误。
```java
/**
 *      描述:     演示使用newFixedThreadPool出现OOM错误
 */
public class FixedThreadPoolOOM {
    // ...
    private static Runnable task() {
        return () -> {
            try {
                Thread.sleep(10000000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
    }
}
```

这里主要就是把任务的睡眠时间加长, 让线程池中的线程消费速度完全更不上生产速度, 就可以引发OOM了。

在运行的时候, 我们可以把JVM内存调小一点(使用: -Xmx3m -Xms3m), 方便测试。


**newSingleThreadExecutor(单一线程池创建)**  
创建该线程池, 我们甚至连参数都不用传递了, newSingleThreadExecutor直接在底层给我们创建好了。我们先来看看newSingleThreadExecutor的使用。

```java


/**
 *         描述:      newSingleThreadExecutor线程池使用例子
 */
public class singleThreadExecutorTest {
    public static void main(String[] args) {
        ExecutorService executorService =
                Executors.newSingleThreadExecutor();
        for (int i = 0; i < 1000; i++) {
            executorService.execute(task());
        }
    }
}
```

task()方法和FixedThreadPoolTest一样都是输出线程名称, 后面的例子都一样, 不在说明。

运行后会发现, 输出的线程名称永远是同一个, 没有其它线程执行。这是为什么呢?
我们点击newSingleThreadExecutor方法, 看看底层实现原理。

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

是不是一目了然, 基本和我们固定线程池原理基本一样, 只不过核心线程和最大线程参数值都是1, 这也就是为什么没有其它线程去执行, 只有一个线程去执行的原因了。


**newCachedThreadPool(可缓存线程池创建)**  
该线程池的底层原理和上面两个就有点不同了, 首先该线程池会创建很多线程来处理任务, 并且会在一定时间内进行回收。

那会创建多少个线程? 多长时间回收? 队列是有界还是无界呢?

我们先通过下面的一个实例运行后, 通过底层源码在来进行回答上面的问题。

```java

/**
 *      描述:     缓存线程池使用
 */
public class CachedThreadPoolTest {
    public static void main(String[] args) {
        ExecutorService executorService =
                Executors.newCachedThreadPool();
        for (int i = 0; i < 1000; i++) {
            executorService.execute(task());
        }
    }
}
```

运行输出结果, 发现竟然有很多很多个线程去执行, 这是为什么呢? 我们看看newCachedThreadPool创建方法的原理。

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

什么! 核心线程数竟然是0, 也就是说之后的线程都会被回收, 时间是1分钟。而且队列使用的是SynchronousQueue这是一个没有容量的队列, 直接进行交互。所以当我们的任务进来后, 就会创建一个新的线程去执行, 而我们的最大线程数是Integer的最大值几乎不会被创建满格的...

这种没有限制的去创建线程, 如果线程数量非常多也是会出现OOM的。

**newScheduledThreadPool(定时&周期线程池创建)**  
该线程池比较上面有点特殊, 它可以等待用户指定的时间去执行任务并且可以周期性的去执行任务。

```java
/**
 *      描述:     调度线程池使用
 */
public class ScheduledThreadPoolTest {

    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService =
                Executors.newScheduledThreadPool(5);

        // 1秒之后执行任务
//        scheduledExecutorService.schedule(task(), 1, TimeUnit.SECONDS);

        // 初始化为1s执行之后, 每次等待3s后再一次执行
        scheduledExecutorService.scheduleAtFixedRate(task(), 1, 3, TimeUnit.SECONDS);
    }
}
```

这里的任务提交和上面所有的都不同, 是使用schedule来去调度执行的。

我们看一下newScheduledThreadPool实现原理

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```

主要就是看一下DelayedWorkQueue, 进入该队列的任务只有达到了指定的延时时间，才会执行任务。其实这里也可看到最大线程数是Integer的最大值, 依旧可能会引发OOM。


经过上面的了解, 使用JDK自带的方法去创建线程池可能会导致线上OOM的可能, 所以我们最好手动的去创建线程池来避免此类问题。

并且手动创建线程池, 可以根据业务来设置线程池参数, 设置对应的线程池名称等。

至于如何合理的设置线程池中的线程数, 这一块我引用一些链接吧。毕竟都是大同小异的。主要还是分: 计算密集任务/IO密集任务。

[手把手教你手动创建线程池](https://juejin.im/post/5e58e0a2f265da574f3541cd)

[如何合理地估算线程池大小?](http://ifeve.com/how-to-calculate-threadpool-size/)

不过这也仅仅是一个参考项, 更多的还是需要更具业务以及环境自行去测试得到一个比较好的参数配置。


##### 线程池停止
上面既然已经创建出来线程池了, 那也可以停止我们的线程池。(这TM不是废话?)
停止线程池有两个方法:
  * shuwdown
  * shutdownNow

并且还包含三个辅助方法, 用来检查线程池是否停止:
  * isShutDown
  * isTerminated
  * awaitTermination

对于上面这5个方法, 我们会进行一个具体例子演示。

```java

/***
 *      描述:     线程池关闭
 */
public class ShutDown {

    public static void main(String[] args) throws InterruptedException {
        /***
         *      既然是演示线程池的关闭, 这里就随意创建一个线程池进行演示.
         */
        ExecutorService executorService =
                Executors.newFixedThreadPool(10);

        for (int i = 0; i < 1000; i++) {
            executorService.execute(task());
        }
        Thread.sleep(1000);
        System.out.println(executorService.isShutdown());
        executorService.shutdown();
        executorService.execute(task());
        System.out.println(executorService.isShutdown());
    }
}
```

调用shutdown方法后并不是直接就将线程池关闭, 而是等待正在执行任务和已存储在队列中的任务结束后才会关闭, 在此期间, 新提交的任务不会被接收的, 会抛出异常信息。

再此期间, 可以通过isShutdown方法来判断线程池是否关闭。但是此方法并不知道线程池中的任务是否已经都执行完毕。


为此, 我们可以使用isTerminated方法来进行判断。

```java
System.out.println(executorService.isTerminated());
Thread.sleep(10000);
System.out.println(executorService.isTerminated());
```

可以观察到两次的输出会不一样, 我们在线程池任务没结束之前输出是false, 等待一定时间后, 输出结果为true, 而此时的线程池也已经终止了。

对于awaitTermination方法来说, 它会等待一定时间后去执行。再此期间进入阻塞状态。

```java
boolean b = executorService.awaitTermination(3, TimeUnit.SECONDS);
// boolean b = executorService.awaitTermination(10, TimeUnit.SECONDS);
System.out.println(b);
```

那么, 还剩下最后的一个方法shutdownNow, 该方法比较暴力了, 直接停止线程池, 中断正在执行的任务, 并返回队列任务集合。

```java
List<Runnable> runnables = executorService.shutdownNow();
```

##### 线程池拒绝策略
无论是人还是机器, 终究还是有极限的。当到达一定的量级时候, 我们就无法处理多出来的任务, 此时就需要拒绝新添加的任务。

对于线程池提供了4种拒绝策略, 分别是:
  * Abort Policy(抛出异常)
  * Discard Policy(直接丢弃)
  * Discard-Oldest Policy(丢弃队列中最老的任务)
  * Caller-Runs Policy(将任务分给调用线程来执行)


下面, 我们会给出每个拒绝策略的具体实现, 当然也可以自定义拒绝策略只需要实现RejectedExecutionHandler接口即可。这块有兴趣的同学自行实现当做练习。

[java-rejectedexecutionhandler](https://www.baeldung.com/java-rejectedexecutionhandler)

```java
/***
 *
 *      描述:     AbortPolicy拒绝策略使用, 抛出异常
 */
public class AbortPolicyTest {
    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor =
                new ThreadPoolExecutor(3, 3, 60, TimeUnit.SECONDS, new SynchronousQueue<>());
        threadPoolExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());

        /**
         *      可以看到, 当线程池处理不过来的时候, 就会抛出java.util.concurrent.RejectedExecutionException异常。
         *      其实, 线程池默认就是使用此策略...
         *
         *      感觉多此一举了...
         */

        for (int i = 0; i < 100; i++) {
            int finalI = i;
            threadPoolExecutor.execute(() -> {
                System.out.println(Thread.currentThread().getName() + finalI + " run ...");
            });
        }
    }
}
```


```java

/***
 *      描述:     CallerRunsPolicy拒绝策略使用, 将任务分给调用线程来执行
 */
public class CallerRunsPolicyTest {
    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor threadPoolExecutor =
                new ThreadPoolExecutor(3, 3, 60, TimeUnit.SECONDS, new SynchronousQueue<>());
        threadPoolExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());


        /***
         *      可以看到, 这里是由main线程提交的任务, 所以交给main线程来执行
         */
        for (int i = 0; i < 100; i++) {
            threadPoolExecutor.execute(() -> {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println(Thread.currentThread().getName() + " 正在执行...");
            });
        }

        Thread.sleep(10000);

        /***
         *      我们创建了一个Thread-A线程提交任务, 就使用Thread-A线程执行任务
         */
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                threadPoolExecutor.execute(() -> {
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + " run...");
                });
            }

        }, "Thread-A").start();
    }
}
```

```java
/***
 *      描述:     DiscardPolicy拒绝策略使用, 直接丢弃任务
 */
public class DiscardPolicyTest {
    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor threadPoolExecutor =
                new ThreadPoolExecutor(1, 1, 0, TimeUnit.SECONDS,  new SynchronousQueue<>());
        threadPoolExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());


        /***
         *      由于生产任务太多, 消费完全更不上。所以会导致后面任务都被丢弃掉
         */

        for (int i = 0; i < 10; i++) {
            threadPoolExecutor.execute(() -> {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println(Thread.currentThread().getName() + " run ...");
            });
        }


        /***
         *  如果在这里等待一定时间后, 线程池有可以使用的线程了, 下面的queue是可以offer进去的,
         *  如果线程池中的所有线程还在执行任务, 这个任务依旧是没有执行的机会, 队列为空
         */
        Thread.sleep(3000);

        BlockingQueue<String> queue = new LinkedBlockingDeque<>();
        threadPoolExecutor.execute(() -> {
            queue.offer("Discarded Result");
            System.out.println("Go...");
        });

        // 这里也需要等待一定时间, 线程池线程不一定offer进去了, 等待后, 程序正常可以看到队列长度是1
        Thread.sleep(1000);
        System.out.println("thread addWork queue size : " + queue.size());
    }
}
```


```java
/***
 *      描述:     DiscardOldestPolicy拒绝策略使用, 丢弃队列中最老的任务
 */
public class DiscardOldestPolicyTest {

    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor threadPoolExecutor =
                new ThreadPoolExecutor(1, 1, 0, TimeUnit.SECONDS,  new ArrayBlockingQueue<>(2));
        threadPoolExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardOldestPolicy());


        /***
         *      现在我们的任务队列大小为2, 有一个核心线程执行。我们需要添加4个任务去执行, 会有下面的情况发生:
         *          1. 第一个任务将单线程占据500毫秒
         *          2. 执行程序成功地将第二个和第三个任务排队
         *          3. 当第四个任务到达时，丢弃最旧的策略将删除最早的任务，以便为新任务腾出空间
         *
         *          所以, 下面的queue只会有[Second, Third], 而First是最早提交的, 所以被移除了。
         *
         *          注意:
         *              丢弃最早的策略和优先级队列不能很好地配合使用。
         *              因为优先级队列的头具有最高优先级，所以我们可能会简单地失去最重要的任务。
         */

        threadPoolExecutor.execute(() -> {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        BlockingQueue<String> queue = new LinkedBlockingDeque<>();
        threadPoolExecutor.execute(() -> queue.offer("First"));
        threadPoolExecutor.execute(() -> queue.offer("Second"));
        threadPoolExecutor.execute(() -> queue.offer("Third"));

        Thread.sleep(1000);
        System.out.println(queue);

    }
}
```

观察一下, 我们使用的队列都是有界的, 或者是直接交互的类型。想象一下如果换成无界的队列会是什么后果? 除非你猝死在工位上, 否则没有人知道你很累, 懂了吗😏


##### 线程池钩子方法
对于钩子方法, 这里简单的描述一下什么是钩子方法。更多了解请大家自行搜索。

**钩子方法: 是一个抽象类提供空实现，子类进行选择性重写的方法**

没错, 线程池也给我们提供了一些钩子方法, 提供方法如下:
  * beforeExecute(线程执行之前)
  * afterExecute(线程执行结束后)
  * terminated(线程池结束)

如果我们想在线程执行之前或者之后记录相关日志, 添加一些包装方法等其他公共一些操作, 我们可以实现这几个方法。

```java
/***
 *      描述:     线程池钩子方法的使用
 */

public class HookThreadPool extends ThreadPoolExecutor {
    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        System.out.println(t.getName() + " 执行之前运行");
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        System.out.println(Thread.currentThread().getName() + " 执行之后运行");
    }

    @Override
    protected void terminated() {
        super.terminated();
        System.out.println("线程池结束后执行...");
    }

    public static void main(String[] args) throws InterruptedException {
        HookThreadPool hookThreadPool =
                new HookThreadPool(5, 10, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(100));

        for (int i = 0; i < 100; i++) {
            hookThreadPool.execute(() -> {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println(Thread.currentThread().getName() + " 开始运行了...");
            });
        }
//        Thread.sleep(5000);
//        hookThreadPool.shutdown();
        hookThreadPool.shutdownNow();
    }
}
```

对于钩子方法的使用, 不仅仅就是一个输出这么简单, 可以根据具体业务逻辑来实现, 这里演示主要是让大家了解有这么一个功能。



##### 线程池纵览
最后呢, 我们会把线程池大体架构以及线程池生命周期等功能进行介绍。

###### 线程池组成部分
线程池由一下四个部分组成:
  * 线程池管理器
    * 用于创建并管理线程池, 包括创建线程池, 销毁线程池, 添加任务等;(Thread-Pool)
  * 工作线程
    * 线程池中的线程, 在没有任务时处于等待状态, 可以循环执行任务;(Thread)
  * 任务队列
    * 用于存放没有处理的任务; (Blocking Queue)
  * 任务接口
    * 每个任务必须实现的接口, 以供工作线程调度任务并执行; (task, 实现Runnable接口的task)

###### ThreadPoolExecutor还是Executors还是Executor?
经过上面的使用, 我们开头介绍了ThreadPoolExecutor相关参数以及基本原理, 但是后期确是使用Executors创建的线程池, 返回的却是ExecutorService类?这些都是什么鬼?

我们先来看一下ThreadPoolExecutor类的关系。

![线程池整体结构](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84.png?raw=true)

明显就可以看到其继承的关系了。这里呢也对这些类做个简单的概括:
  * Executor是最基础的接口, 其中只包含了execute()方法
  * ExecutorService继承Executor, 并在其上增加了shutdown,submit等方法扩展。

  * AbstractExecutorService抽象类实现了ExecutorService接口中的大部分方法
  * TheadPoolExecutor继承了AbstractExecutorService，是线程池的具体实现

其中, 当我们使用周期性的线程池即newScheduledThreadPool返回的是ScheduledExecutorService, 其也是ExecutorService子类。

Executors是线程池的静态工厂，其提供了快捷创建线程池的静态方法。

###### 线程池实现任务复用
线程池是如何让线程不停歇的处理我们的提交的任务, 线程池的线程不也是线程吗?
难道不会终止吗?

带着疑问, 我们主要看一看ThreadPoolExecutor类下的addWorker()方法和其私有子类Worker类下的runWorker()方法。

当我们在执行execute()方法时候, 会进入下面代码段
```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

我们不用关注其它判断, 这些是线程池我们最开始说的参数判断, 我们只关注addWorker()方法。

```java
private boolean addWorker(Runnable firstTask, boolean core) {
  // ....

  boolean workerStarted = false;
  boolean workerAdded = false;
  Worker w = null;
  try {
      w = new Worker(firstTask);
      final Thread t = w.thread;
      if (t != null) {
          final ReentrantLock mainLock = this.mainLock;
          mainLock.lock();
          try {
              int rs = runStateOf(ctl.get());

              if (rs < SHUTDOWN ||
                  (rs == SHUTDOWN && firstTask == null)) {
                  if (t.isAlive()) // precheck that t is startable
                      throw new IllegalThreadStateException();
                  workers.add(w);
                  int s = workers.size();
                  if (s > largestPoolSize)
                      largestPoolSize = s;
                  workerAdded = true;
              }
          } finally {
              mainLock.unlock();
          }
          if (workerAdded) {
              // 启动我们的线程
              t.start();
              workerStarted = true;
          }
      }
  } finally {
      if (! workerStarted)
          addWorkerFailed(w);
  }
  return workerStarted;
}
```

addWorker()方法前置一些判断, 我们并不关心, 我们只关注当前展示这部分, 通过addWorker()方法会启动一个线程任务, 并将Worker任务添加至HashSet中。

由于在addWorker()方法中, 线程会被启动。而启动的线程就是Worker类包装后的任务,所以我们来看看Worker类, 它实现了Runnable接口, 也是一个工作线程。所以当addWorker()方法调用了start()方法时, 会执行Worker类的run()方法, run()方法中就是调用runWorker()方法。

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        /*
          这里之所以可以重复执行正式因为此循环, 不断去检测有没有任务,
          如果worker中的任务不为空，继续执行，否则使用getTask获得任务。一直死循环，除非得到的任务为空才退出
        */
        while (task != null || (task = getTask()) != null) {
            w.lock();
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 这里就会调用我们任务的run方法, 就和调用普通方法一样
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```
runWorker()方法执行逻辑:
  1. 使用一个变量获取当前提交的任务判断是否为空, 如果不为空执行firstTask, 之后置为空;

  2. firstTask为空后调用getTask()方法从队列中获取任务执行;

  3. 一直循环执行任务, 直到没有可执行任务, 退出while循环;

所以, 通过这里的循环不断去判断还有没有任务没有被执行, 如果有, 就会一直利用已有的线程去处理我们提交的任务, 等到队列中的任务全部被执行完后, 退出循环。

这样就实现使用相同的线程去处理不同的任务, 反复执行新任务。从而达到复用的条件。

想详细了解, 可以参考这篇文章

[Java线程池ThreadPoolExecutor源码分析](https://fangjian0423.github.io/2016/03/22/java-threadpool-analysis/)

[彻底弄懂 Java 线程池原理](https://juejin.im/post/5c33400c6fb9a049fe35503b)

###### 线程池生命周期
线程有生命周期, 线程池自然也有自己的生命周期, 线程池的状态又有多少种? 又是如何转换的?


我们先进入ThreadPoolExecutor类的源码来看看有哪些状态
```java
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

线程池有五种状态, 分别对应如下:
  * RUNNING: 接收新的任务并处理队列中的任务
  * SHUTDOWN: 不接收新的任务，但是处理队列中的任务
  * STOP: 不接收新的任务，不处理队列中的任务，同时中断处理中的任务
  * TIDYING: 所有的任务处理完成，有效的线程数是0
  * TERMINATED: terminated()方法执行完毕


通过下图, 我们来看一下线程池状态的一个转换过程

![线程池状态](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%8A%B6%E6%80%81.png?raw=true)

[Java线程池的5种状态](https://blog.csdn.net/GoGleTech/article/details/79728522)

[Java线程池ThreadPoolExecutor使用和分析(一)](https://www.cnblogs.com/trust-freedom/p/6594270.html)

[Java线程池ThreadPoolExecutor使用和分析(二) - execute()原理](https://www.cnblogs.com/trust-freedom/p/6681948.html)
