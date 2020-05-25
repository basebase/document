#### Java多线程核心6-线程属性

##### Java线程属性概览
我们在创建线程的时候, 每个线程都有自己唯一的id, 也可以为线程设置一个名字, 或者设置线程的优先级以及判断一个线程是否为守护线程。

对于线程的名字我们可能不会很默认了, 经过前面这么多的学习和了解, 但是对于线程的优先级以及守护线程可能不是特别了解。

我们优先来看看线程id和线程名称, 最后在看看守护线程以及线程的优先级。

##### 线程id以及线程名称

###### 线程ID
在进入线程ID之前呢, 我们先思考下面两个问题:
  * 线程ID是从0开始还是从1开始?
  * 当我们创建的子线程ID是我们最初设想的递增ID吗?


```java

/***
 *      描述:     线程ID从1开始, 但是JVM运行起来后, 我们自己创建的线程ID早已不是2
 */

public class ThreadID {
    public static void main(String[] args) {
        Thread thread = new Thread();


        /***
         *      private static synchronized long nextThreadID() {
                    return ++threadSeqNumber;
                }

                这个是线程id的逻辑, threadSeqNumber初始化为0, 但是由于++在前面
                所以是先加1然后在返回, 所以当我们的mian线程启动运行ID值为1
         */
        System.out.println("线程 " + Thread.currentThread().getName() + " ID: "
                + Thread.currentThread().getId());


        /***
         *     线程id既然是自增的, 那么我们自己创建的子线程为什么不是2呢? 而是其它的数值了呢?
         *     当我们debug程序(进入idea的threads界面, 选中main线程右键选中Export Threads), 会发现在启动子线程之前JVM就已经启动了其它线程了
         *          * Finalizer@
         *          * Reference Handler@
         *          * Signal Dispatcher@
         *     我们会发现这些线程后面都有一个@符号, 其实就是对应的ID, 我们也可以看到main@1信息
         *     所以, 这就是我们的子线程为什么后续的线程ID不是2的原因。
         *
         */
        System.out.println("线程 " + thread.getName() + " ID: "
                + thread.getId());
    }
}
```

###### 线程名称
在进入线程名称之前呢, 我们先思考下面两个问题:
  * 创建线程时不设置名称, 可以动态修改线程名称吗?
  * 我们动态更新的Java线程名称可以更新到JVM本地方法的线程名称吗?

```java
/***
 *      描述:     线程名称, 如果不给定一个名称后, 默认会用Thread-(ID)设置名称, ID从0开始
 */
public class ThreadName {
    public static void main(String[] args) throws InterruptedException {

        /***
         *      可以看到, 在初始化的时候如果我们不设置名称, 会默认写入"Thread-" + nextThreadNum()
         *      来看看nextThreadNum()方法
         *      private static synchronized int nextThreadNum() {
                    return threadInitNumber++;
                }
         */

        for (int i = 0 ; i < 10; i ++) {
            Thread thread = new Thread();
            thread.start();
            System.out.println(thread.getName());
        }

        Thread loveThread = new Thread();
        System.out.println(loveThread.getName());

        /***
         *      可以在创建线程的时候不给定一个线程名称, 通过调用setName()方法设置线程名称
         *      setName()方法有一个name还有一个setNativeName()方法, 我们先来说说name就是我们Java线程的名称
         *      即使Java的线程启动了, 我们依然可以修改线程的名称。
         *      
         *      而想要设置NativeName, 只能在线程还没有start之前设置, setNativeName是一个本地方法
         *
         *      this.name = name;
         *      if (threadStatus != 0) {
                    setNativeName(name);
                }
         */
        loveThread.setName("love");
        loveThread.start();
        System.out.println(loveThread.getName());
    }
}
```


##### 守护线程以及线程优先级

###### 守护线程

线程一般分两类:
  * 用户线程
  * 守护线程

而我们创建的线程默认一般都是用户线程, 如果需要显示的把用户线程转为守护线程需要调用setDaemon()函数。

那么, 守护线程有什么特点?平时写的一些测试程序有守护线程的存在吗?

假设当前我们的程序只有一个main线程以及N个守护线程, 当我们的main线程结束后, 无论这些守护线程有没有运行完毕, JVM都会退出。但是, 如果我们除了main线程还有其余的用户线程还在执行中, 此时JVM不会退出, 而是等待其余的用户线程结束之后, JVM才会退出。

**也就是说守护线程无法影响JVM左右。**

一般我们随便写的任意带有main方法的java类, 都是有守护线程的, 比如我们的垃圾收集器Finalizer等线程。


```java

/***
 *      描述:     设置守护线程晚于main线程之前, 验证用户线程结束守护线程是不是也退出了
 */
public class ThreadDaemon {
    public static void main(String[] args) throws InterruptedException {


        Thread daemonThread = new Thread(() -> {
            int count = 0;
            while (count++ < 1000) {
                try {
                    System.out.println(Thread.currentThread().getName() + " : " + count);
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "daemonThread");

        /**
         *      设置线程为守护线程
         */
        daemonThread.setDaemon(true);
        daemonThread.start();

        System.out.println(Thread.currentThread().getName() + " 线程开始等待中...");
        Thread.sleep(5000);

        /***
         *     可以看到当main线程结束睡眠的同时, 我们的程序就结束了, 而不在意我们的守护线程有没有执行完毕
         */

        System.out.println(Thread.currentThread().getName() + " 线程结束了...");
    }
}
```


###### 线程优先级

java线程优先级分为1~10, 数字越大代表线程优先级越高。默认创建的线程是5(其实不然。默认的优先级是父线程的优先级。在init方法里有下面一小段代码)。

```java

// Thread.java

private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {

      // ...
      Thread parent = currentThread();
      this.priority = parent.getPriority();
}
```

或许这么解释是因为Java程序的主线程(main方法)的优先级默认是为NORM_PRIORITY，这样不主动设定优先级的，后续创建的线程的优先级也都是NORM_PRIORITY了。


但是, 需要注意的是, 我们的设计不能依赖线程的优先级, 这是因为不同的操作系统优先级会被操作系统改变。具体可以参考这篇文章对比每个系统下的优先级

[什么是Java 线程优先级？](https://www.javamex.com/tutorials/threads/priority_what.shtml)

[java优先级无效](https://stackoverflow.com/questions/12038592/java-thread-priority-has-no-effect)

[线程的优先级](https://www.cnblogs.com/duanxz/p/5226109.html)

对于优先级来说, 不建议去修改, 用默认的优先级即可。所以这里就了解大概的一个意思即可。
