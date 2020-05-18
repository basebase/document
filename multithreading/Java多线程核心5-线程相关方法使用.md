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

##### 使用wait/notify实现生产消费模型
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
