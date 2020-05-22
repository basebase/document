#### Java多线程核心5-线程相关方法使用

##### Java多线程相关方法使用
本小结主要是学习线程相关API, 并且也是对线程生命周期的一个深入学习。通过学习相对于的API可以
更清晰的知道线程之间的转换。

那么, 可以操作线程相关的方法一个是Thread类另外一个是Object类。  
通过下面的表格, 我们看看有那些API需要我们进行学习。

|        类 |                      方法 |
|     :-----|                    :---- |
|  Thread   |       sleep              |
|  Thread   |       join               |
|  Thread   |       yield              |
|  Thread   |   currentThread          |
|  Thread   |       start              |
|  Thread   |   stop()/suspend/resuem  |
|  Thread   |   interrupt              |
|  Object   |   wait/notify/notifyAll  |

这里有很多方法我们其实已经学习并且已经使用过了, 所以我们重点要关注的API是wait/notify/notifyAll相关方法, 这也是本小结学习的一个重点。

在使用相关API方法的时候, 我会对每个方法进行一个描述。


##### wait/notify/notifyAll相关方法使用

wait()方法有以下特点:
  * 必须在持有锁的情况下调用wait()方法, 否则会抛出异常
  * 调用wait()方法会释放所持有的对象锁(monitor)
  * 线程进入等待
  * 如果设置超时时间则自动被唤醒(如果设置超时时间为0则永久等待), 否则需要被其它线程唤醒


notify()方法有以下特点:
  * 必须在持有锁的情况下调用notify()方法, 否则会抛出异常
  * 唤醒一个等待的线程(如果有多个线程需要被唤醒, 那么选择唤醒哪一个线程是不确定的)

notifyAll()方法有以下特点:
  * 必须在持有锁的情况下调用notifyAll()方法, 否则会抛出异常
  * 唤醒所有等待线程, 但是哪一个先处理取决于系统


有了上面对两个方法的宏观认识之后, 我们通过具体的代码实现看看wait()方法和notify()的使用方法

###### wait/notify配合使用例子
```java
/**
 *     描述:      wait/notiyf配合使用
 *               1. 线程执行顺序
 *               2. wait释放锁
 */
public class WaitNotify {

    public static void main(String[] args) throws InterruptedException {

        Object obj = new Object();

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (obj) {
                    System.out.println("线程 " + Thread.currentThread().getName() + " 开始执行wait()方法");
                    try {
                        // 释放锁并且进入等待状态
                        // 当然, 如果执行了中断也是会抛出异常
                        obj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println("线程 " + Thread.currentThread().getName() + " 执行结束.");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (obj) {
                System.out.println("线程 " + Thread.currentThread().getName() + " 执行notify()方法");
                obj.notify();
                System.out.println("线程 " + Thread.currentThread().getName() + " 执行结束.");
            }
        });

        t1.start();

        /***
         * 中间设置一下间隔, 先让线程t1执行这样线程t2执行notify()方法才是一个有效的方法,
         * 否则没有一个线程进入等待状态, 就算执行了也是一个空的
         */
        Thread.sleep(500);

        t2.start();


        /***
         *      线程执行流程如下:
         *             1. 线程T1启动后, 并执行相关代码, 当执行到wait()方法后, 释放锁(monitor)。
         *                如何证明释放了锁(monitor)? 很简单, 如果线程T1没有释放锁(monitor), 线程T2不可能执行
         *
         *             2. 线程T1进入等待状态后, 线程T2开始执行, 并且唤醒其中一个"等待状态"中的线程, 让其可以再此执行
         *                但是, 由于当前obj的锁已经被线程T2所持有了, 所以线程T1必须要等待线程T2执行完成才有机会获取到锁(或者线程T2调用wait()方法)
         *
         *             3. 当线程T2执行完后, 线程T1继续开始往下执行。
         *
         *             即: T1 -> T2 -> T1 -> EXIT
         */

        //
        /***
         * 如果在没有同步代码块的位置调用wait()方法呢?
         * 则直接抛出java.lang.IllegalMonitorStateException
         *
         */
        Thread.sleep(1000);
        Thread t3 = new Thread(() -> {
            System.out.println("线程 " + Thread.currentThread().getName() + " 非同步代码块执行wait");
            try {
                obj.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        t3.start();
    }
}
```


###### wait/notifyAll配合使用例子

```java
/***
 *      描述:         wait/notifyAll配合使用, 当前有三个线程A,B,C
 *                   A和B线程进入阻塞, 通过C线程唤醒A和B线程
 */
public class WaitNotifyAll {

    private static Object obj = new Object();

    public static void main(String[] args) throws InterruptedException {
        Runnable r = task();

        Thread threadA = new Thread(r);
        Thread threadB = new Thread(r);

        Thread threadC = new Thread(() -> {
            synchronized (obj) {
                System.out.println(Thread.currentThread().getName() + " 调用notifyAll()方法");
                obj.notifyAll();

                /***
                 *   如果这里使用了notify()方法的话, 那么程序不会终止, 始终会有一个线程一直是等待状态
                 *   毕竟notify()方法只能唤醒一个线程。
                 */
//                obj.notify();
            }
        });


        /***
         *     执行说明:
         *         1. 执行步骤①和步骤②, 此时线程被启动, 至于谁优先执行取决系统调度。
         *
         *         2. 如果将步骤③注释掉, 可能会出现线程A或者线程B其中一个落后于线程C执行, 或者线程A和线程B都落后线程C执行
         *            这就会导致线程C的唤醒是无效的, 程序不会被终止。一直是等待状态。
         *
         *         3. 如果步骤③没有被注释的话, 则会顺利的唤醒线程A和线程B, 两个线程去获取锁进而执行后面的程序。
         */

        // ①
        threadA.start();
        // ②
        threadB.start();
        // ③
        Thread.sleep(1000);
        // ④
        threadC.start();
    }

    public static Runnable task() {
        return () -> {
            synchronized (obj) {
                System.out.println(Thread.currentThread().getName() + " 开始执行任务.");
                try {
                    // 模拟正在执行任务, 休眠1s
                    Thread.sleep(300);
                    System.out.println(Thread.currentThread().getName() + " 开始释放锁进入阻塞.");
                    obj.wait();
                    System.out.println(Thread.currentThread().getName() + " 从新获取到锁, 执行完成.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
    }
}
```


###### wait只释放当前monitor例子

```java


/***
 *      描述:     证明wait()只释放当前的锁, 锁之间相互独立。
 */

public class WaitNotifyReleaseOwnMonitor {


    /***
     *     该测试类有两个线程A和B, 然后线程A优先获取到resourceA和resourceB, 并释放resourceA锁
     *     同时线程B获取到resourceA的锁, 但是却无法获取resourceB的锁, 证明每个锁都是独立的。
     */
    private static class WaitNotifyReleaseOwnMonitorTest1 {

        private static Object resourceA = new Object();
        private static Object resourceB = new Object();

        public void run() {
            Thread threadA = new Thread(() -> {
                synchronized (resourceA) {
                    System.out.println(Thread.currentThread().getName() + " 获取到resourceA锁");
                    try {
                        // 模拟执行任务
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    synchronized (resourceB) {
                        System.out.println(Thread.currentThread().getName() + " 获取到resourceB锁");
                        try {
                            // 释放resourceA
                            System.out.println(Thread.currentThread().getName() + " 释放resourceA锁");
                            resourceA.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                }
            });


            Thread threadB = new Thread(() -> {

                try {
                    // 等待线程A释放锁
                    Thread.sleep(500);

                    synchronized (resourceA) {
                        System.out.println(Thread.currentThread().getName() + " 获取到resourceA锁");

                        System.out.println(Thread.currentThread().getName() + " 尝试获取resourceB锁");
                        synchronized (resourceB) {
                            System.out.println(Thread.currentThread().getName() + " 获取到resourceB锁");
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });


            threadA.start();
            threadB.start();
        }

    }


    /***
     *      该类利用三个线程A,B,C进行测试, 线程A获取resourceA锁并将锁释放进入阻塞, 线程B获取resourceB锁并将锁释放进入阻塞,
     *      然后线程C证明两把锁是独立的, 毕竟resourceA锁和resourceB锁, 它们所持有的线程都不一样。
     *
     *      事实证明, 如果在resourceA锁上调用notifyAll()方法, resourceB锁上的线程不会被其唤醒。程序还是在等待状态。反之亦然。
     *      除非, resourceA和resourceB两把锁都调用notifyAll()才可将其上的线程都唤醒。
     */
    private static class WaitNotifyReleaseOwnMonitorTest2 {
        private static Object resourceA = new Object();
        private static Object resourceB = new Object();

        public void run() {

            Thread threadA = new Thread(() -> {
                synchronized (resourceA) {
                    System.out.println(Thread.currentThread().getName() + " 获取到resourceA锁");
                    try {
                        // 模拟任务执行
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + " 释放resourceA锁");
                    try {
                        resourceA.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + " 结束任务.");
                }
            });


            Thread threadB = new Thread(() -> {
                synchronized (resourceB) {
                    System.out.println(Thread.currentThread().getName() + " 获取到resourceB锁");
                    try {
                        // 模拟任务执行
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + " 释放resourceB锁");
                    try {
                        resourceB.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println(Thread.currentThread().getName() + " 结束任务.");
                }
            });

            Thread threadC = new Thread(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                /**
                 *      为了证明锁之间是相互独立的, 这里我们用锁resourceA唤醒线程, 看看线程B是否也会将其唤醒
                 */
                synchronized (resourceA) {
                    System.out.println(Thread.currentThread().getName() + " 唤醒resourceA锁上的线程");
                    resourceA.notifyAll();
                }

//                synchronized (resourceB) {
//                    System.out.println(Thread.currentThread().getName() + " 唤醒resourceB锁上的线程");
//                    resourceB.notifyAll();
//                }

                /***
                 * 使用this相当于又是一把锁, 如果在this对象中直接调用notifyAll()方法, 则一个线程都不会被唤醒
                 * 毕竟this锁中没有一个线程进入了等待状态
                 */
//                synchronized (this) {
//                    notifyAll();
//                }
            });

            threadA.start();
            threadB.start();
            threadC.start();

        }
    }



    public static void main(String[] args) {
//        WaitNotifyReleaseOwnMonitorTest1 waitNotifyReleaseOwnMonitorTest1 =
//                new WaitNotifyReleaseOwnMonitorTest1();
//        waitNotifyReleaseOwnMonitorTest1.run();

        WaitNotifyReleaseOwnMonitorTest2 waitNotifyReleaseOwnMonitorTest2 =
                new WaitNotifyReleaseOwnMonitorTest2();
        waitNotifyReleaseOwnMonitorTest2.run();
    }
}
```


##### wait/notify/notifyAll常见面试问题

###### 使用wait/notify实现生产消费模型
通过上面几个例子, 大概对wait/notify/notifyAll已经有一定的认识了, 也知道如何去应用了。  
那么, 我们通过wait/notify来实现一个生产消费者模型, 当我们生产者达到一定的上限后就会阻塞不在生产数据, 当我们的消费者没有数据可消费时候进入阻塞等待生产者生产数据。


```java

/***
 *      描述:         用wait/notify实现生产消费模型
 */
public class ProducerConsumerModel {

    private Object obj = new Object();
    // 生产者写入此数据结构中, 消费者读取此数据结构中数据
    private LinkedList<Integer> storage = new LinkedList<>();
    private static final Integer CONTAIN = 100;
    private static final Integer MAX_SIZE = 10;


    /***
     *            生产者线程任务:
     *                 生产者任务上限是10条数据, 当超过后就会进入阻塞状态, 需要其它线程将其唤醒。
     */
    public Runnable producerTask() {
        return () -> {
            for (int i = 0; i < CONTAIN; i ++) {
                synchronized (obj) {
                    try {
                        // 如果队列已满, 进入阻塞
                        if (storage.size() == MAX_SIZE)
                            obj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    storage.add(i);
                    System.out.println(Thread.currentThread().getName() + " 生产数据 : " + i + " 加入队列队列大小 : " + storage.size());
                    // 队列已经有数据了, 唤醒消费者消费数据
                    obj.notify();
                }
            }
        };
    }


    /***
     *            消费者线程任务:
     *                 消费者消费数据, 如果当前队列数据为空则进入阻塞, 需要其它线程将其唤醒
     */
    public Runnable consumerTask() {
        return () -> {
            for (int i = 0; i < CONTAIN; i ++) {
                synchronized (obj) {
                    try {
                        // 如果没有数据可以消费, 则进入阻塞等待生产者将其唤醒
                        if (storage.size() == 0)
                            obj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " 消费数据 : " + storage.poll() + " 队列还剩下 " + storage.size() + " 元素没有被消费");
                    // 队列现在有空闲, 唤醒生产者生产数据
                    obj.notify();
                }


            }
        };
    }

    public static void main(String[] args) {
        ProducerConsumerModel producerConsumerModel2 = new ProducerConsumerModel();
        Thread producerThread = new Thread(producerConsumerModel2.producerTask(), "producer");
        Thread consumerThread = new Thread(producerConsumerModel2.consumerTask(), "consumer");

        producerThread.start();
        consumerThread.start();

    }
}
```


**注意: 如果我们把producerTask和consumerTask两个方法的synchronized放在最外层会有什么问题**

```java
synchronized (obj) {
    for (int i = 0; i < CONTAIN; i ++) {
        ...
    }
}
```

具体代码参考ProducerConsumerModel2.java

你会发现, 每次都是生产者堆满了之后唤醒消费者, 消费者消费完了唤醒生产者。先不要着急往下看, 自己先思考一下为什么?

聪明的你肯定想到啦, 必然是这样啊, synchronized放在最外层, 就算我们的生产者或者消费者唤醒正在阻塞的线程, 但是此时锁是在某一个线程上的, 只有等待持有锁的线程进入阻塞另外一个线程才能获取到锁进而执行。所以就看到生产者满了才执行消费者, 消费者读取完队列数据才能执行生产者。

而synchronized放在for循环内, 如果消费或者生产线程被唤醒, 在进行下一次循环都有机会获取到锁, 就会看到交叉的打印结果集。


###### 两个线程交替打印奇偶数

现在有两个线程, 假设A线程输出奇数B线程输出偶数。进行一个交替的输出。  
如:Thread-A:1,Thread-B:2,hread-A:3,Thread-B:4...

该问题可以用synchronized来解决, 也可以使用wait/notify来解决。两种解决方式wait/notify是对synchronized的完善, 解决synchronized造成的问题。

具体原因已经在代码注释中写入了, 这里不再重复。


```java
/**
 *      描述:     两个线程交替打印0~100的奇偶数, 使用synchronized来实现
 */
public class WaitNotifyPrintOddEvenSync {

    private static int count;
    private static final Integer SIZE = 100;
    private static Object obj = new Object();

    public static void main(String[] args) {

        /***
         *      1. 创简两个线程, 一个输出奇数一个输出偶数
         *      2. 通过synchronized来完成交替
         *
         *      下面的两个线程可以正确的输出奇偶数, 但是有以下问题:
         *              1. 由于synchronized是在while循环体内的, 所以当结束synchronized代码块线程又可以抢占需要的锁
         *              2. 哪个线程能抢到锁资源是不确定的, 所以当奇数线程正确输出后应该是偶数线程输出,但是锁却被奇数线程再次抢占
         *              3. 这就导致线程有很多空操作
         *
         */

        // 偶数线程
        new Thread(() -> {
            while (count < SIZE) {
                synchronized (obj) {  // 可能当前是偶数, 但是抢不到锁资源导致无法执行
                    System.out.println("===================> even");
                    if (count % 2 == 0) {
                        System.out.println(Thread.currentThread().getName() + " : " + count);
                        count ++;
                    }
                }
            }
        }, "even").start();

        // 奇数线程
        new Thread(() -> {
            while (count < SIZE) {
                synchronized (obj) {
                    System.out.println("===================> odd");
                    if (count % 2 == 1) {
                        System.out.println(Thread.currentThread().getName() + " : " + count);
                        count ++;
                    }
                }
            }
        }, "odd").start();
    }
}
```


```java
/**
 *
 *      描述:     两个线程交替打印0~100的奇偶数, 使用wait/notify来实现
 */
public class WaitNotifyPrintOddEvenWait {

    private static int count;
    private static final Integer SIZE = 100;
    private static Object obj = new Object();

    public static void main(String[] args) throws InterruptedException {

        /***
         *
         *      1. 两个线程共用同一个任务, 无论哪个线程获取到锁都直接打印出内容。
         *      2. 打印完后, 唤醒其它线程, 当前线程休眠
         *
         *      为什么可以这么做?
         *          假设count从0开始, 我们偶数线程先行, 打印完后进入休眠, 这样奇数线程就能获取到锁
         *          然后奇数线程打印完后进入休眠, 接着偶数线程运行...依次运行, 直到count > SIZE结束任务
         *
         */

        Runnable runnable = task();
        /***
         *  这样的启动方式没问题, 但是有可能会串位, 谁也不知道哪个线程优先执行, 所以要保证正确的话,
         *  可以使用sleep来阻塞一下下
         */
        new Thread(runnable, "even").start();
        Thread.sleep(100);
        new Thread(runnable, "odd").start();

    }

    public static Runnable task() {
        return () -> {
            while (count < SIZE) {
                synchronized (obj) {
                    // 当前线程输出对应的值
                    System.out.println(Thread.currentThread().getName() + " : " + count++);
                    // 必须要唤醒上一次等待的线程
                    obj.notify();

                    // 如果count还小于SIZE就必须进入等待状态, 等待另外一个线程将其唤醒
                    if (count < SIZE) {
                        try {
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        };
    }
}
```


###### wait方法为什么需要在synchronized(同步代码块)中使用, 而sleep不需要?
比如现在有两个线程A和线程B, 我们要求线程A释放锁并且进入阻塞状态, 而线程B去唤醒所有正在等待被唤醒的线程, 我们分别使用start方法启动线程, 假设优先执行了线程A但是还没有执行到wait方法, 而此时切换到了线程B执行notify/notifyAll方法, 这个时候线程A执行到wait方法后, 由于线程B已经执行完了所以线程A会一直处于等待的状态。

而通过synchronized可以有效的避免此类问题的发生, 当线程A执行中并没有释放锁, 线程B是无法执行的, 只有等待线程A释放锁进入等待状态后, 线程B获取到相关锁后才可以执行对应方法。

而sleep是当前线程, 并不需要和其它线程配合使用。所以不需要放在synchronized中。


###### wait/notify/notifyAll为什么在Object中?而sleep在Thread中?

1. 在Java中，为了进入临界区代码段，线程需要获得锁并且它们等待锁可用，它们不知道哪些线程持有锁而它们只知道锁是由某个线程保持，它们应该等待锁而不是知道哪个线程在同步块内并要求它们释放锁。 这个比喻适合等待和通知在object类而不是Java中的线程

2. 每个对象都可以作为锁，这是另一个原因wait和notify在Object类中声明，而不是Thread类。


###### 使用Thread.wait会怎么样?
可以用, 但是有一个问题, Thread类比较特殊, 线程在退出的时候会自动执行notif方法, 这样会导致整体逻辑出现问题。

```java


/**
 *      描述:     如果使用Thread类的wait方法, 当线程执行并退出会调用notify/notifyAll方法
 */
public class ThreadExitCallBackNotify {

    public static void main(String[] args) {

        Thread lockThread = new Thread(lockThread());
        Object lock = new Object();

        /***
         *      一个是不同的Object对象, 一个是Thread实例, 使用Thread线程在退出的时候会调用notify方法
         */
        Runnable runnable = taskN1(lockThread);
//        Runnable runnable = taskN1(lock);
        Thread t1 = new Thread(runnable);
        Thread t2 = new Thread(runnable);

        t1.start();
        t2.start();
        lockThread.start(); // 但是, 如果线程不调用start方法的话, 依旧和普通的对象锁一样, 不会调用notify方法
    }

    public static Runnable lockThread() {
        return () -> {
            System.out.println("lockThread start ...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("lockThread end ...");
        };
    }

    public static Runnable taskN1(Object lock) {
        return () -> {
            synchronized (lock) {
                System.out.println(Thread.currentThread().getName() + " start ...");
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " end ...");
            }
        };
    }
}
```



##### sleep方法详解

sleep方法可以称得上是我们的老朋友了, 一直在使用。但是对于sleep方法大家又知道多少呢？

sleep方法可以放线程进入阻塞状态, 并释放CPU资源, 但不释放锁, 直到超出休眠时间后再执行,
休眠期间如果被中断, 会抛出异常并清除中断状态。

sleep的特性:
  1. 让线程进入阻塞状态, 不占用CPU资源
  2. 不释放当前线程的锁, 无论是synchronized还是Lock

对于第一点我们不做过多介绍, 当我们调用sleep后, 线程都会让出当前执行的CPU, 让其它线程可以有机会执行, 当然如果需要拥有锁的代码块的话, 即使当前线程sleep其它线程也无法执行相关代码。

而我们重点要介绍的是第二点, sleep不会释放锁, 这一点上和我们的wait()方法有所不同。
我们通过两个实例来验证。


```java

/***
 *      描述:     展示sleep不释放锁, 等待sleep超时后, 正常退出synchronized代码块才释放锁
 */
public class SleepDontReleaseMonitor {

    public static void main(String[] args) {
        SleepDontReleaseMonitor sleepDontReleaseMonitor = new SleepDontReleaseMonitor();
        Runnable runnable = sleepDontReleaseMonitor.take();

        /***
         *      可以看到线程在等待5s后继续执行, 退出synchronized代码块后另外一个线程才能进入synchronized代码块
         */
        new Thread(runnable).start();
        new Thread(runnable).start();
    }

    public Runnable take() {
        return () -> {
            synchronized (this) {
                System.out.println(Thread.currentThread().getName() + " 获得到锁.");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 即将释放锁.");
            }
        };
    }
}
```


```java

/***
 *      描述:        演示sleep不释放lock(需要手动释放)
 */

public class SleepDontReleaseLock {
    private Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        SleepDontReleaseLock sleepDontReleaseLock = new SleepDontReleaseLock();
        Runnable runnable = sleepDontReleaseLock.take();
        new Thread(runnable).start();
        new Thread(runnable).start();
    }

    public Runnable take() {
        return () -> {
            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName() + " 获得到lock.");
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(Thread.currentThread().getName() + " 释放lock.");
                lock.unlock(); // 这里如果不手动释放lock另外一个线程永远都是阻塞状态
            }
        };
    }
}
```


###### sleep常见面试问题

wait/notify和sleep异同?

相同点如下:
  * 阻塞
  * 响应中断


不同点如下:
  * 同步代码块(wait/notify必须在持有锁的情况下调用, 否则抛出异常)
  * 释放锁(wait会释放锁, sleep不会)
  * 规定时间(sleep必须传入参数, wait可以不需要)
  * 所属类(wait/notify属于Object, sleep属于Thread)



##### join方法详解

其实说白了就是一个线程等待另外一个线程, 这就是join的功能。

比如我们要让main线程等线程A执行完成之后才执行后续逻辑。在比如说, 我们现在有5条线程初始化资源, 我们需要等待这5个线程初始化完成后再继续执行后续逻辑。


###### join方法实践

join方法示例, 用main线程等待其它子线程执行完成后, 在执行后续逻辑。

```java

/***
 *      描述:     join例子展示
 */
public class Join {

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 执行完成");
        });

        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 执行完成");
        });

        /***
         *      执行顺序如下:
         *          1. 首先输出 main 开始执行
         *          2. 线程0和线程1启动
         *          3. 等待线程1和线程0执行完成后继续执行main线程后续逻辑
         */

        System.out.println(Thread.currentThread().getName() + " 开始执行");
        t1.start();
        t2.start();

        // 如果我们把t2.join(), t1.join()注释后, 那么输出顺序可能就是main线程全部输出来了, 然后在输出线程0和线程1的内容
        t2.join();
        t1.join();

        System.out.println(Thread.currentThread().getName() + " 执行完成");
    }
}
```

join期间线程被中断

```java


/***
 *      描述:     join期间被中断的演示
 */
 public class JoinInterrupt {

    public static void main(String[] args) {

        Thread mainThread = Thread.currentThread();

        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 开始执行...");
            try {

                /***
                 *      在子线程中调用主线程main的中断方法, main线程正在等待子线程Thread-0
                 */
                mainThread.interrupt();
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName() + " 执行结束...");
        });

        System.out.println(Thread.currentThread().getName() + " 开始");

        t1.start();
        try {

            /***
             *      当子线程调用主线程的中断方法, 被中断的是主线程
             *      如果我们没有把异常信息传递给线程0的话, 子线程0还是一个阻塞状态等待5s后才结束
             *      如果我们调用线程0的中断方法后, 可以立即终止子线程0
             */
            t1.join();
        } catch (InterruptedException e) {
            System.out.println(Thread.currentThread().getName() + " 被中断了.");
            e.printStackTrace();
            t1.interrupt(); // 如果我们注释掉此语句后, 可以看到程序不会立即终止, 而是按照子线程原有逻辑时间输出。
        }

        System.out.println(Thread.currentThread().getName() + " 结束");
    }
}
```

当线程等待另外一个线程时, 会进入WAITING状态

```java

/**
 *      描述:     main线程在join中是什么状态?
 */
public class JoinThreadState {
    public static void main(String[] args) throws InterruptedException {
        Thread mainThread = Thread.currentThread();
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(3000);
                System.out.println(Thread.currentThread().getName() + " 开始执行...");
                System.out.println(mainThread.getName() + " 线程状态: " + mainThread.getState());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });

        t1.start();
        /**
         *      当mian线程等待子线程执行的时候会进入WAITING(等待)状态, 等子线程执行完后进入RUNNABLE状态
         */
        t1.join();
        System.out.println(Thread.currentThread().getName() + " 线程状态 : "
                + Thread.currentThread().getState());
    }
}
```
