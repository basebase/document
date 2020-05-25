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
