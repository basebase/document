#### Java多线程核心3-线程中断

##### 线程中断介绍

##### 前言
当我们开发了一c款GUI界面程序会应用到多线程, 比如点击某个杀毒软件的取消按钮来停止查杀病毒, 当我们想停止某个文件下载或者是实现了多线程的任意程序。我们想要中断某一个线程正在执行的任务, 都需要通过一个线程去取消另外一个线程正在执行的任务。Java没有提供一种安全直接的方法来停止某个线程。但是Java提供了中断机制。

如果对Java中断没有一个全面了解, 可能会误认为被中断的线程会立马退出运行。但事实并非如此。我们带着以下几个疑问去学习:

* 中断机制是如何工作的？
* 捕获或检测到中断后, 是抛出InterruptedException还是重设中断状态以及在方法中吞掉中断状态会有什么后果?
* Thread.stop与中断相比有哪些异同?
* 什么情况下使用中断?


##### 中断原理

**Java中断机制是一种协作机制, 也就是说通过中断并不能直接终止另一个线程, 而需要被中断的线程自己处理中断。**
怎么理解这段话呢?

假设: 当线程t1想中断线程t2, 只需要在t1中将线程t2对象的中断标识置为true, 然后线程t2可以选择在合适的时候处理该中断请求, 甚至可以不处理中断请求, 不处理中断请求的线程就和没有被中断一样。

也就是说Java提供了一种"通知"的功能, 具体被通知的线程是否按照要求处理中断或者不处理中断我们无法控制, 所以称为一种协作的机制。

简单了解一下相关API, 这里只是简单的一些描述, 最好还是参考相关API, 尤其是interrupt()方法, 比较详细。
[JDK API](https://docs.oracle.com/javase/8/docs/api/)

| 方法 | 描述 |
|                               :-----| :---- |
| public static boolean interrupted() | 测试当前线程是否已经中断。线程的中断状态 由该方法清除换句话说，如果连续两次调用该方法，则第二次调用将返回 false|
| public boolean isInterrupted()      | 测试线程是否已经中断。线程的中断状态不受该方法的影响。 |
| public void interrupt()             |   中断线程。 |


##### 线程中断实例
上面的基本原理和基本API我们已经大概了解了线程中断是什么意思, 但是具体如何去做呢?
我们通过具体的例子来揭Java线程中断神秘的面纱吧...

推荐实战例子参考:
  * [Interrupting a Thread](https://www.javatpoint.com/interrupting-a-thread)
  * [How a thread can interrupt an another thread in Java?
](https://www.geeksforgeeks.org/how-a-thread-can-interrupt-an-another-thread-in-java/)
  * [When does Java's Thread.sleep throw InterruptedException?
](https://stackoverflow.com/questions/1087475/when-does-javas-thread-sleep-throw-interruptedexception)

###### 不带阻塞, 仅仅中断一个线程

```java
/***
 *      描述:     run方法内没有sleep或wait方法时停止线程。
 */
public class RightWayStopThreadWithoutSleep implements Runnable {

    @Override
    public void run() {
        int num = 0;
        // 在不加入Thread.currentThread().isInterrupted()判断和没事人一样。
        while (!Thread.currentThread().isInterrupted() && num <= Integer.MAX_VALUE / 2) {
            if (num % 10000 == 0)
                System.out.println(num + "是10000的倍数");
            num ++;
        }

        System.out.println("任务运行结束...");
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new RightWayStopThreadWithoutSleep());
        t1.start();

        // 用来等待1s后再进行中断
        Thread.sleep(1000);
        // main线程中断t1线程
        t1.interrupt();
    }
}
```

其中, interrupt方法是唯一能将中断状态设置为true的方法。静态方法interrupted会将当前线程的中断状态清除, 而我们使用isInterrupted方法来进行检测是否中断线程。

上面的例子中, main线程通过调用interrupt方法将线程t1的中断状态设置为true, 线程t1可以在合适的时候调用interrupted或者isInterrupted方法来检测并做相应的处理。当然, 也可以不对中断状态做任何处理。


###### 带有阻塞(sleep)的中断
假设, 我们在线程中加入了阻塞(sleep)方法呢?执行线程中断又会是什么结果?

```java

/***
 *      描述：     run方法带有sleep的中断线程的写法
 */
public class RightWayStopThreadWithSleep {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
          try {
              int num = 0;
              while (num <= 300 && !Thread.currentThread().isInterrupted()) {
                  if (num % 100 == 0)
                      System.out.println(num + "是100的倍数");
                  num ++;
              }
              Thread.sleep(2000);
              System.out.println(Thread.currentThread().getName() + " 线程结束");
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
        };

        Thread t1 = new Thread(runnable);
        t1.start();

        // 等待1s让线程t1优先执行
        Thread.sleep(1000);
        // t1线程休眠2s, main线程执行中断线程, 但是t1处于阻塞状态
        // 那么t1退出阻塞状态并抛出一个java.lang.InterruptedException异常
        t1.interrupt();

        System.out.println(Thread.currentThread().getName() + " 线程结束");
    }
}
```
该程序最终会退出阻塞并抛出一个异常信息。


###### 循环迭代每次sleep的中断
```java
/**
 *      描述:         在执行线程中循环调用sleep或者wait等方法(可以不判断当前线程是否中断状态)
 */
public class RightWayStopThreadWithSleepEveryLoop {
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            try {
                int num = 0;
                while (num < 1000 /* && !Thread.currentThread().isInterrupted() */) {
                    System.out.println("当前的值为: " + num);
                    num ++;
                    Thread.sleep(10);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        Thread t1 = new Thread(runnable);
        t1.start();

        t1.sleep(5000);
        t1.interrupt();
    }
}
```
每次循环迭代中都调用sleep方法可以不需要在循环中判断是否需要中断。当我们调用线程的中断方法时线程处于阻塞状态会退出阻塞状态并抛出一个异常中断线程。所以这和检查是否中断状态无关, 而是当中断遇到阻塞的时候就会退出阻塞抛出异常。(最前面推荐的实战例子也有对应的英文介绍)


###### 循环迭代捕获sleep方法
上面都是try/catch循环体, 下面的例子中, 如果我们只try/catch sleep方法呢?
会有什么异同?

```java
/***
 *      描述：     while体内加入try/catch, 会导致中断失效
 */
 public class CanInterrupt {
     public static void main(String[] args) throws InterruptedException {
         Thread t1 = new Thread(getRunnable());
         t1.start();
         Thread.sleep(5000);
         t1.interrupt();
     }

     public static Runnable getRunnable() {
         return () -> {
             int num = 0;
             // ①    加入线程中断状态也无效, 已经被sleep清空了状态
             while (num < 100000 /* && !Thread.currentThread().isInterrupted()*/) {
                 System.out.println("当前值为: " + num);
                 try {
                     Thread.sleep(300);
 //                    System.out.println("当前值为: " + num);   ②   不会输出
                 } catch (InterruptedException e) {
                     e.printStackTrace();

 //                    int i = 1 / 0;                           ③   抛出异常后终止
 //                    return ;                                 ④   退出
                 }

                 num ++;
             }
         };
     }
 }
```

线程会抛出异常, 但可以发现线程没有被中断。还一直在执行, 直到大于while条件或者手动退出。这是为什么呢?

我们设置让线程中断, 被中断的线程就不应该在继续执行了啊!里应如此, 而且确实也抛出了异常信息。这里我们有两个点: (try/catch) + sleep。

① sleep可以清空线程的中断状态


② 如果对异常不是很了解的同学可能需要稍微了解一下, 当发生异常, 我们try/catch住后,try内容后面是不会执行的而是进入我们的catch块。而我们的catch没有抛出其它新的异常或者return, 所以后面的代码依旧可以运行。也就是要满足while退出条件。(参考: [java抛出异常后代码继续执行的情况](https://blog.csdn.net/anne_IT_blog/article/details/76926920))

当我们第一次调用了线程中断, 确实触发了异常。异常被捕获catch并没有做任何其它工作。而我们的sleep已经清空了中断状态了。那么, 这种情况下如何处理呢?


###### 线程中断的异常抛出问题

当我们提供一个方法应用到线程的同时, 我们需要注意在处理异常的时候不要吞掉异常信息, 而是在方法中抛出来让线程方法去捕获异常, 这样在线程中断的时候就能感知到, 能在异常的同时写入对应的事件。


<font color="#60C66C">1: 传递中断(优先选择)</font>
```java
/***
 *      描述:     try/catch捕获InterruptedException后
 *               优先选择: 在方法签名中抛出异常, 那么在run()方法就会强制try/catch
 */
public class RightWayStopThreadInProd {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(getRunnable());
        t1.start();

        // 休眠1s后, t1线程休眠2s, 这样就能触发阻塞异常
        Thread.sleep(1000);
        t1.interrupt();

    }

    public static Runnable getRunnable() {
        return () -> {
            int num = 0;
            while (num < 10000) {
                System.out.println("当前值为: " + num);
                num ++;

                // ① 调用异常方法, 但是该方法自行处理了异常, 并没有抛出任何异常信息到run方法, 而是吞掉了异常信息 (不推荐)
//                throwInMethod1();


                // ② 推荐在方法签名上抛出异常信息, 这样run()方法可以捕获到异常,
                //    注意run()方法里面只能使用try/catch, 不能使用throws, 这是因为其父类方法并没有任何异常抛出。 (推荐)
                try {
                    throwInMethod2();
                } catch (InterruptedException e) {
                    // 响应中断信息
                    // 保存日志, 记录状态等等
                    System.out.println("当前状态已记录...");
                    e.printStackTrace();
                }
            }
        };
    }

    /***
     *      直接try/catch异常后, 并不做任何处理。直接吞掉异常信息。(不推荐)
     */
    private static void throwInMethod1() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /***
     * 在方法签名中抛出异常 。(推荐)
     * @throws InterruptedException
     */
    private static void throwInMethod2() throws InterruptedException {
        Thread.sleep(2000);
    }
}
```


<font color="#60C66C">2: 无法传递中断</font>

比如我们的run方法或者我们自己写入的方法想自己处理异常信息, 而不是抛出去。
当然这也是可以的。

```java

/***
 *      描述:     在catch中调用Thread.currentThread().interrupt()来恢复设置中断状态。
 *               以便在后续的执行中, 依然能够检查到刚才发生了线程中断。(解决RightWayStopThreadInProd中断问题)
 */
public class RightWayStopThreadInProd2 {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(getRunnable());
        t1.start();

        // 休眠1s后, t1线程休眠2s, 这样就能触发阻塞异常
        Thread.sleep(1000);
        t1.interrupt();

    }

    public static Runnable getRunnable() {
        return () -> {
            int num = 0;
            while (num < 10000) {

                // 检查线程是否中断
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("响应线程中断, 退出中...");
                    break ;
                }

                System.out.println("当前值为: " + num);
                num ++;

                // ①    不用处理异常信息, 方法已经处理了, 并重新设置了中断信息
//                reInterrupt();

                // ②    在run方法中自己处理异常信息, 重新设置中断信息
                try {
                    throwInMethod();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    e.printStackTrace();
                }
            }
        };
    }

    /***
     * 捕获异常信息, 不吞并异常信息, 重新设置中断信息。
     */
    private static void reInterrupt() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        }
    }

    private static void throwInMethod() throws InterruptedException {
        Thread.sleep(2000);
    }
}
```
该程序可以正确的中断线程, 程序不会再继续执行了。

经过这么多例子, 总结一下几个点:



  1. [线程调用调用其它方法优先使用传递中断的方式](#线程中断的异常抛出问题)
  2. [线程调用调用其它方法无法传递中断, 不要吞并而是恢复中断](#线程中断的异常抛出问题)
  3. [循环迭代, 异常信息被吞并, 线程响应中断但还是继续执行](#循环迭代捕获sleep方法)



##### interrupt vs stop
可以看到stop方法在JDK中已经被弃用了, 已经不推荐使用了。为什么呢?  
如果我们使用stop方法终止线程, 那么被终止的线程会直接停止, 这就会造成一个后果
产生脏数据。

使用stop方法会释放锁。

比如交易系统, 你给我转账100元, 但是实际只有10元, 但是认为转账成功。  
在比如说, 我们给一个班级中的小组分配作业, 有10个小组, 其中只有3个小组被分配到作业, 其余的小组没有。而这个时候我们也以为分配成功了。

这些问题会给数据造成不一致性。而且排查问题也特别的麻烦。  
我们通过下面的例子来看看stop运行的结果。

可以参考Oracle给出的文档, 为什么弃用了stop, suspend和resume相关方法。  
[Java Thread Primitive Deprecation](https://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html)

```java


/**
 *     描述:      使用stop()方法停止线程, 会导致线程运行到一半就停止, 没办法完成一个基本单位的操作
 *               比如说: 假设一个班级中有10个小组, 每个小组都需要领取自己的作业, 如果使用stop()停止线程可能会出现有的学生没有领取到自己的作业
 */
public class StopThread {

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(getRunnable());
        t1.start();
        Thread.sleep(2000);
        t1.stop();
    }

    public static Runnable getRunnable() {
        return () -> {

            /***
             *
             *  这里, 就是我们的一个基本的执行块。
             *      然而, 当线程调用了stop()方法时候, 而stop()方法不会抛出异常而是一个Error子类ThreadDeath。
             *      通过结果可以看到, stop()停止线程后, 一个班级下领取到作业, 有的没有。
             *      而我们之前使用interrupt()方法, 可以在catch中做一些善后操作, 比如回滚数据等。
             *      而stop()方法完全没有任何商量余地, 直接就GG了。所以, 不推荐使用stop()方法
             *
             */

            for (int i = 0; i < 10; i ++) {
                System.out.println("第 " + i + " 个班级下");
                for (int j = 0; j < 10; j ++) {
                    System.out.println("====> 第 " + i + " 个班级下第 " + j + " 个小组领取作业完成");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        System.out.println("会执行我catch吗?");
                        e.printStackTrace();
                    }
                }
            }

            System.out.println("exit.");
        };
    }
}
```


##### 使用volatile停止线程?
通常, 我们看到有很多人利用一个volatile设置一个变量值, 通过读取变量值来判断是否中断线程。  
那么, 通过volatile进行中断线程, 有没有问题呢?

我们通过两个例子来看看volatile存在的一个问题。

```java

/***
 *      描述:     演示使用volatile用来停止线程,
 */
public class WrongWayVolatile {

    private volatile boolean canceled = false;

    public static void main(String[] args) throws InterruptedException {
        WrongWayVolatile wrongWayVolatile = new WrongWayVolatile();
        Runnable runnable = wrongWayVolatile.getRunnable();
        Thread t1 = new Thread(runnable);

        t1.start();
        Thread.sleep(5000);
        wrongWayVolatile.canceled = true;
    }

    public Runnable getRunnable() {
        return () -> {
            try {
                int num = 0;
                while (num < 100000 && !canceled) {
                    System.out.println("当前的值为: " + num);
                    Thread.sleep(10);
                    num ++;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println("线程终止...");
            }
        };
    }
}
```

上述的例子中, 可以正常的停止掉一个线程, 看着没有任何问题。

但是, 假设如果我们要停止的线程被阻塞了呢? 线程被挂起, 通过volatile变量能终止线程吗?
我们利用阻塞队列来模拟这种情况。当队列满了, 线程就会被挂起。这个时候如果我们修改volatile
变量状态, 能否终止线程呢?

```java

/***
 *      描述:     使用volatile停止线程, 当遇到阻塞还能停止吗?
 */

public class WrongWayVolatileCantStop2 {
    public static void main(String[] args) throws InterruptedException {
        BlockingQueue storage = new ArrayBlockingQueue(10);
        Producer p = new Producer(storage);

        Thread producerThread = new Thread(p);
        producerThread.start();

        Thread.sleep(3000);
        Consumer c = new Consumer(storage);
        while (c.needMoreNums()) {
            System.out.println(c.storage.take() + "被消费了 !");
            Thread.sleep(100);
        }

        System.out.println("消费完成...");
        // 当消费完后, 我们更新生产者的状态, 让其停止线程执行
        p.canceled = true;

        System.out.println("状态: " + p.canceled);
        System.out.println("当前生产者量级: " + p.storage.size());
    }
}


class Producer implements Runnable {

    public volatile boolean canceled = false;
    BlockingQueue storage;

    public Producer(BlockingQueue storage) {
        this.storage = storage;
    }

    @Override
    public void run() {
        try {

            int num = 0;
            while (num < 100000 && !canceled) {
                System.out.println("当前值: " + num + " 放入队列中了!");

                /***
                 *      当队列满了之后, 线程就会再这里被挂起, 如果没有被唤醒, 那么就算我们的canceled变量更新
                 *      当前的while判断也是读取不到的, 从而无法终止当前的循环条件, 而是会一直阻塞再此。
                 */
                storage.put(num); // 当队列满了, 线程在这里就会被挂起
                Thread.sleep(10);
                num ++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("生产者结束运行");
        }
    }
}

class Consumer {
    BlockingQueue storage;

    public Consumer(BlockingQueue storage) {
        this.storage = storage;
    }

    public boolean needMoreNums() {
        if (Math.random() > 0.95)
            return false;
        return true;
    }
}
```

可以看到, 程序是没有被终止的, 这是因为"生产者(Producer)"发现队列已经满了, 不能再写入, 进而阻塞线程, 然而, 虽然我们的main线程更新的volatile的状态值, 但是, 线程被阻塞无法进行while的判断条件, 所以导致线程不能立即响应中断信号。

如果对阻塞队列不是很熟悉的话, 可以参考另外一个例子, 使用sleep模拟阻塞超长时间
[WrongWayVolatileCantStop.java](https://github.com/basebase/java-examples/blob/master/src/main/java/com/moyu/example/multithreading/ch03/WrongWayVolatileCantStop.java)

那么, 上述的程序如何修复呢? 其实很简单, 依旧是使用线程中断的方法, 可以让阻塞中的线程抛出中断异常信息。参考源码[WrongWayVolatileFixed.java](https://github.com/basebase/java-examples/blob/master/src/main/java/com/moyu/example/multithreading/ch03/WrongWayVolatileFixed.java)

**经过上面的两个例子, volatile看似可以用来终止一个线程, 但是如果遇到线程阻塞却没有办法立即终止线程。**

##### 响应中断方法列表
我们只对sleep方法做了中断的响应, 可能在中断之前做什么其他操作等之类的。
那么除了sleep方法还有哪些方法可以响应中断信息呢?

|  类 | 方法 |
|     :-----|                              :---- |
|  Object   |   wait()/wait(long)/wait(long, int)|
|  Thread   |   sleep()/sleep(long)/join()/join(long)/join(long, int)        |
|  BlockingQueue   | take()/put(E) |
|  Lock   |   lockInterruptibly()                |
|  CountDownLatch   |   await()                  |
|  CyclicBarrier   |   await()                   |
|  Exchanger   |   exchange(V)                   |
|  InterruptibleChannel   |                      |
|  Selector   |                      |


##### 总结

本节内容:
  * 如何正确的停止线程(使用interrupt()方法), 反观stop/suspend/resume先关方法已经被弃用。而使用volatile来停止线程遇到线程阻塞则无法中断线程。

  * 线程中断, 异常信息不能吞掉, 而是向上抛出异常或者处理异常信息。
  


参考:
  1. [处理 InterruptedException](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)
  2. [详细分析 Java 中断机制](https://www.infoq.cn/article/java-interrupt-mechanism)
  3. [Java里一个线程调用了Thread.interrupt()到底意味着什么？](https://www.zhihu.com/question/41048032)
