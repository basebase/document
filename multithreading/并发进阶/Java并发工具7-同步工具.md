### Java并发工具7-同步工具

#### 概述
在学习多线程的时候, 我们可以使用join()方法来等待某个线程执行完后再执行。虽然join可以实现线程之间的同步但是join()方法也是有缺点的。

1. 需要获取到线程的引用调用join()
2. 当多个线程需要控制顺序执行join()方法会显得比较乱

当我们使用juc下面的工具包来控制线程之间的同步顺序, 多个线程使用同一个实例管理也很方便。

下面, 我们先来看看JUC给我们提供了那些线程之间同步的工具类

| 类 | 说明 |
| :-----:| :----: | 
| CountDownLatch | 多个线程相互等待, 当计数器为0时, 线程被释放 |
| CyclicBarrier | 线程之间相互等待, 当计数器为0时, 释放线程后可重新使用 |
| Semaphore | 线程在拥有信号量的"许可证"才可以执行 |
| Condition | 控制线程的等待和唤醒 |
| Phaser |  |
| Exchanger |  |


#### CountDownLatch使用

CountDownLatch类使用其实非常简单, 当我们进入CountDownLatch源码就会发现更本就没有几个方法。但是我们最主要的关注的方法有三个。

1. 构造方法只有只有, 并且需要传入等待的数量

```java
public CountDownLatch(int count) {
    //...
}
```


2. 计算器减1, 假设我们传入计数器的值为5, 那么就需要执行该方法5次, 否则等待中的线程永远都不会往下执行
```java
public void countDown() {
    // ...
}
```

3. 线程等待, 当构造方法中传入的count数量值不为0时, **哪个线程调用此方法, 哪个线程就一直是阻塞状态, 只有当count值为0才会被唤醒执行**

```java
public void await() throws InterruptedException {
    // ...
}
```


通过了解上面的API我们也知道了个大概, 当我们在主线程中调用了await()方法时, 子线程如果没有进行countDown()方法的话, 主线程就会一直阻塞, 当子线程们都执行countDown()直到0时, 就会唤醒等待中的主线程。

**这里需要注意一点: 只要没调用await()方法的线程都不会被阻塞, 多个线程调用会同时被阻塞。**


**样例1: 现在我们要去提交OA审批(申请一台显示器), 期间可能会有很多人给我们审批, 等全部审批通过之后我们才可以去运维部门领取显示器, 只要期间有一个人审批不通过我们就会一直阻塞住无法继续执行。**

```java

/***
 *
 *      描述:     CountDownLatch使用, 让主线程等待其余子线程执行完成后, 在执行。
 */
public class CountDownLatchExample01 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(5);
        ExecutorService executorService =
                Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int id = i + 1;
            executorService.execute(() -> {
                System.out.println("用户ID: " + id + " 正在查阅任务...");
                int r = new Random().nextInt(6) + 1;
                System.out.println("用户ID: " + id + " 正在审批任务, 大约耗时: " + r + "秒");
                try {
                    Thread.sleep(r * 1000);
                    System.out.println("用户ID: " + id + " 审批完成");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    latch.countDown();
                }
            });
        }

        System.out.println("提交任务主线程用户正在默默等待结果中...");
        latch.await();
        System.out.println("提交任务主线程用户审核通过啦!!!");

        executorService.shutdown();
    }
}
```

该例子中, 当中只要有一个线程没有执行latch.countDown()方法都会导致mian线程陷入无尽的阻塞中。


**样例2: 现在有一群学生需要等老师来了发放试卷然后开始做题, 但是在发放试卷顺序可能会造成有些同学优先做题有些学生晚一点做题, 这样就会造成时间上的不公平, 我们要做的是只有当老师说现在开始做题之后大家才能一起动笔写题。**

```java
/***
 *
 *      描述:     CountDownLatch使用, 让多个子线程等待一个主线程
 */

public class CountDownLatchExample02 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(1);
        ExecutorService executorService =
                Executors.newFixedThreadPool(5);

        for (int i = 0; i < 5; i++) {
            final int id = i + 1;
            executorService.execute(() -> {
                System.out.println("学生编号: " + id + " 领取到试卷, 等待做题...");
                try {
                    latch.await();
                    System.out.println("学生编号: " + id + " 开始做题...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        Thread.sleep(3000);     // 等待一定时间, 老师正在拿着试卷走回教室
        System.out.println("各位同学开始考试...");
        latch.countDown();
        executorService.shutdown();
    }
}
```

此例子中, 我们让所有学生领取到试卷之后才能开始写题。


至此, 对比上面两个样例, 我们发现使用CountDownLatch可以有下面两种方式:
  1. 一个线程等待多个线程执行完毕
  2. 多个线程等待一个线程执行完毕

样例1中就是一个线程等待多个线程执行完成后才能去做后续的事情, 而我们的样例2就不一样了, 它需要等待所有学生都拿到试卷后才能一起做题。

对于无论是一个线程等待多个线程还是多个线程等待一个线程取决于我们的CountDownLatch的构造参数设置, 我们要做的是更具业务逻辑调用countDown()方法将其归零即可。至于在哪里或者什么时候调用countDown()方法取决于我们自己。

对于CountDownLatch类, 我们当然还可以混用, 如下例子:

**样例3: 每个学生写完并提交试卷的时间都是不一样的, 有些人很快有些人很慢, 老师需要等到最后一名同学提交试卷后才会走, 这个时候就需要老师等待学生了。**

```java
/***
 *
 *      描述:     CountDownLatch使用, 多个子线程等待主线程以及主线程等待多个子线程
 */
public class CountDownLatchExample03 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch teach = new CountDownLatch(1);       // 老师的门闩
        CountDownLatch student = new CountDownLatch(5);     // 学生的门闩

        ExecutorService executorService =
                Executors.newFixedThreadPool(5);

        for (int i = 0; i < 5; i++) {
            final int id = i + 1;
            executorService.execute(() -> {
                System.out.println("学生编号: " + id + " 领取到试卷, 等待做题...");
                try {
                    teach.await();
                    System.out.println("学生编号: " + id + " 开始做题...");
                    int r = new Random().nextInt(10) + 1;
                    Thread.sleep(r * 1000);
                    System.out.println("学生编号: " + id + " 提交试卷");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    student.countDown();
                }
            });
        }

        Thread.sleep(3000);
        teach.countDown();  //  开始考试

        student.await();    // 等待学生都提交试卷
        System.out.println("所有学生都提交了试卷....");
        executorService.shutdown();
    }
}
```

此例中可以发现, 学生既要等老师说开始考试才能开始做题, 老师也需要等到最后一个学生提交试卷后才能离开。使用两个CountDownLatch进行配和。

对于CountDownLatch也是有缺点的:
  1. CountDownLatch是不可以复用的, 当我们调用过await()方法后, 如果还是同一实例的CountDownLatch则无效不会等待。


#### Semaphore使用

Semaphore翻译为信号量, Semaphore可以控同时访问的线程个数。什么意思呢? 线程只有拥有Semaphore提供的"许可证"才可以执行, 否则就会进入阻塞状态。

举个例子: 小明, 小黄, 小蓝, 小白, 小青五个人在同一家公司做程序猿, 但是年纪都比较大了要去结婚。由于公司资源紧张, 只有向公司提交申请才能准许请假去结婚(并且名额只有3名)。此时有两种提交方式:

1. 大家谁先提交谁先获取到这个准许名额;
2. 随机提交, 老板优先看到谁的就批谁的;

这个例子中呢, 小明, 小黄, 小蓝, 小白, 小青他们就是我们的线程数量, 而3个名额就是信号量产生的许可证。只有获取到许可证的线程才可以执行, 其余没有获取到的线程就会进入阻塞状态。


现在我们来看看Semaphore中的方法有哪些:

```java
public Semaphore(int permits, boolean fair) {
    // ...
}
```

构造方法中第一个参数是我们的许可证数量, 第二个是公平还是非公平。对于这里还是使用公平的策略。由于限制线程执行数量这个方法中肯定存在长时间的操作(比如上面例子中, 回家结婚就需要比较长时间), 如果使用了非公平插队执行本就需要长时间等待会更加没有执行的机会。


```java
public void acquire() throws InterruptedException {
    // 获取一个许可
}

public void acquire(int permits) throws InterruptedException {
    // 获取permits许可数量, 比如说一次获取3个
}

public void acquireUninterruptibly() {
    // 这个方法不常用, 如果不想被中断使用此方法
}

public boolean tryAcquire() {
    // 尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
}

public boolean tryAcquire(int permits) {
    // 尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
}

public boolean tryAcquire(long timeout, TimeUnit unit)
    // 尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
}

public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
    throws InterruptedException {
    // 尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
}
```

```java
public void release() {
    // 释放一个许可
}

public void release(int permits) {
    // 释放permits个许可
}
```

整体API方法下来是不是和我们学习Lock的API很像, 同样是获取锁, 尝试获取锁并且要释放锁。

```java

/***
 *      描述:     SemaPhore使用例子, 控制线程访问量
 */

public class SemaPhoreExample01 {
    public static void main(String[] args) {

        // 创建信号量
        Semaphore semaphore = new Semaphore(3);

        ExecutorService executorService =
                Executors.newFixedThreadPool(50);

        for (int i = 0; i < 50; i++) {
            final String id = "User-" + (i + 1);
            executorService.execute(() -> {
                System.out.println(Thread.currentThread().getName() + " : " + id + " 申请产假...");
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + " : " + id + " 申请产假成功!");
                    int r = new Random().nextInt(5) + 1;
                    System.out.println(Thread.currentThread().getName() + " : " + id + " 产假休息需要" + r + "天");
                    Thread.sleep(r * 1000);
                    System.out.println(Thread.currentThread().getName() + " : " + id + " 休息产假回来了, 释放许可证...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }

        executorService.shutdown();
    }
}
```

程序运行每次只有三个线程会去执行, 其它线程都会阻塞住。因为许可证的数量只有3个, 被优先提交的三个线程获取到了。

注意点:
 1. 使用SemaPhore如果申请的许可证数量为5, 但是在释放的时候只释放了1个许可证, 即使释放了其余线程依旧无法执行, 因为许可证的数量不够, 还有4个许可证没有被释放掉。

 2. 一般使用公平的调度方式, 使用非公平可能导致任务长时间无法执行。

 3. 释放许可证并非一定要使用已经获取到许可证的线程, 任何线程都可以释放。

 
 ```java
/***
 *      描述:     SemaPhore使用例子, 在非获取许可证的线程中统一释放许可证
 */

public class SemaPhoreExample02 {

    static Semaphore semaphore = new Semaphore(2);

    public static void main(String[] args) {

        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            if (i == 2) {
                threads[i] = new Thread(releaseTask(2), "Thread-" + i);
            } else {
                threads[i] = new Thread(acquireTask(), "Thread-" + i);
            }
        }

        for (int i = 0; i < 5; i++) {
            threads[i].start();
        }
    }

    public static Runnable acquireTask() {
        return () -> {
            System.out.println(Thread.currentThread().getName() + " 尝试获取许可证...");
            try {
                semaphore.acquire(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName() + " 获取到了许可证...");
            // 此任务不释放许可证
        };
    }

    public static Runnable releaseTask(int releaseNum) {
        return () -> {
            // 用来清除积压的许可证任务
            int r = new Random().nextInt(3) + 1;
            System.out.println(Thread.currentThread().getName() + " 正在清除积压任务, 耗时: " + r + "秒");
            try {
                Thread.sleep(r * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            semaphore.release(releaseNum);
            System.out.println(Thread.currentThread().getName() + " 清空积压任务, 又有许可证可以使用了...");
        };
    }
}
 ```
该例子证明可以跨线程去释放我们的许可证, 只要我们在当前线程中没有去执行semaphore.acquire()方法就不会被阻塞, 但是线程可以使用semaphore.release()方法帮助其它线程释放许可证, 其它使用信号量的线程可以再次获取许可证。



#### Condition使用
使用Condition其实和我们的学习使用wait/notify是一样的, 它可以利用await/signal进行等待和唤醒, 而notifyAll对应的则是signalAll方法。

Condition是一个接口, 要创建一个Condition我们需要通过Lock来返回一个Condition实例。为什么要绑定在一个Lock(锁)上面呢?
其实, 大致猜想就能想到, 想要wait/notify是需要获取到锁的, 没有锁的话会抛出异常信息。而通过绑定Lock则可以知道Condition实例对应的是哪个锁, 知道唤醒哪把锁上的线程或者这把锁上有哪些阻塞的线程。

所以, 我们要知道Condition的方法就是下面几个
```java

// 线程进入阻塞状态
void await() throws InterruptedException;
// 唤醒一个线程(唤醒等待时间最长的线程)
void signal();
// 唤醒所有线程
void signalAll();
```

**样例1: 现在有两个线程, 第一个线程会做一些初始化任务, 而第二个线程会在初始化结束后再执行后续任务。通过Condition来实现。**

```java
/***
 *      描述:     使用Condition配合线程阻塞和唤醒
 */
public class ConditionExample01 {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public Runnable task1() {
        return () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 任务初始化中...");
                int r = new Random().nextInt(3) + 1;
                System.out.println(Thread.currentThread().getName() + " 任务初始化预估时间为: " + r + "秒");
                Thread.sleep(r * 1000);
                System.out.println(Thread.currentThread().getName() + " 初始化完成, 唤醒等待任务...");
                condition.signal();     // 唤醒线程
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        };
    }

    public Runnable task2() {
        return () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 等待任务初始化完成...");
                condition.await();      // 阻塞线程
                System.out.println(Thread.currentThread().getName() + " 任务初始化完成, 执行后续任务...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        };
    }

    public static void main(String[] args) throws InterruptedException {
        ConditionExample01 conditionExample01 = new ConditionExample01();
        new Thread(conditionExample01.task2(), "Thread-B").start();
        Thread.sleep(10);       // 让线程Thread-B先执行进行等待, Thread-A优先执行则会出现线程无法被唤醒
        new Thread(conditionExample01.task1(), "Thread-A").start();
    }
}
```

**样例2: 使用Condition实现生产消费者模式**

```java
/***
 *      描述:     Condition生产者消费者模式
 */
public class ConditionExample02 {

    private Lock lock = new ReentrantLock();
    Condition p = lock.newCondition();

    private int size = 10;
    LinkedList<Integer> queue = new LinkedList<>();


    public Runnable producer() {
        return () -> {
            for (int i = 0; i < 100; i++) {
                lock.lock();
                try {
                    if (queue.size() == size) {
                        System.out.println(Thread.currentThread().getName() + " 生产者队列已满, 进入等待...");
                        p.await();
                    }
                    queue.add(i);
                    System.out.println(Thread.currentThread().getName() + " 生产剩余容量为: " + (size - queue.size()));
                    p.signal();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        };
    }

    public Runnable consumer() {
        return () -> {
            for (int i = 0; i < 100; i++) {
                lock.lock();
                try {
                    if (queue.size() == 0) {
                        System.out.println(Thread.currentThread().getName() + " 队列空啦, 进入阻塞");
                        p.await();
                    }

                    Integer v = queue.poll();
                    System.out.println(Thread.currentThread().getName() + " 消费数据为: " + v +
                            " 队列大小为: " + queue.size());

                    p.signal();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        };
    }

    public static void main(String[] args) {
        ConditionExample02 conditionExample02 = new ConditionExample02();
        new Thread(conditionExample02.producer(), "Producer").start();
        new Thread(conditionExample02.consumer(), "Consumer").start();
    }
}
```

通过Condition实现一个生产消费模型, 不过有一个点还是要注意一下, **千万不要把lock.lock()写在了for循环外层, 这样会导致即使线程被唤醒也无法获取到锁, 这个不仅仅是在提示我自己, 也是在提示大家。**


不过使用Condition要比使用wait/notify好很多, 我们可以使用同一个Lock实例创建多个Condition实例对象, 而使用wait/notify则不行。

**样例3: 创建多个Condition实例对象**

```java
/**
 *      描述:     创建多个Condition实例对象
 */
public class ConditionExample03 {

    private Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();


    public Runnable task1() {
        return () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 开始执行任务");
                System.out.println(Thread.currentThread().getName() + " 依赖线程Thread-B进入等待");
                c1.await();
                System.out.println(Thread.currentThread().getName() + " 执行结束");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        };
    }

    public Runnable task2() {
        return () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 开始执行任务");
                System.out.println(Thread.currentThread().getName() + " 依赖线程Thread-C进入等待");
                c2.await();
                System.out.println(Thread.currentThread().getName() + " 执行结束, 唤醒Thread-A");
                c1.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        };
    }

    public Runnable task3() {
        return () -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 开始执行任务");
                System.out.println(Thread.currentThread().getName() + " 执行结束, 唤醒Thread-B");
                c2.signal();
            } finally {
                lock.unlock();
            }
        };
    }

    public static void main(String[] args) throws InterruptedException {
        ConditionExample03 conditionExample03 = new ConditionExample03();
        new Thread(conditionExample03.task1(), "Thread-A").start();
        new Thread(conditionExample03.task2(), "Thread-B").start();
        Thread.sleep(10);
        new Thread(conditionExample03.task3(), "Thread-C").start();
    }
}
```

#### CyclicBarrier使用
CyclicBarrier作用和CountDownLatch功能类似, 等待所有线程就位之后然后一起触发执行。而CyclicBarrier比CountDownLatch不同的一点就是可以循环使用, 当我们设定等待数量为5的时候, CountDownLatch之后就不可以使用了, 而我们CyclicBarrier还可以。

对于CyclicBarrier只需要关注下面几个API即可。

```java

public CyclicBarrier(int parties) {
    // 创建CyclicBarrier对象, 设置等待线程数量
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    // 这个构造方法就比较有意思, 提供了一个Runnable接口, 当所有线程都就位之后, 由最后一个进入的线程执行
}

public int await() throws InterruptedException, BrokenBarrierException {
    // 线程等待
}

public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
            BrokenBarrierException,
            TimeoutException {
    // 有时间的等待, 指定时间过后自行唤醒执行
}
```


**样例1: 等待N个线程初始化完成统一工作**

```java

/**
 *      描述:     CyclicBarrier例子使用
 */
public class CyclicBarrierExample01 {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
        for (int i = 0; i < 5; i++) {
            new Thread(task(cyclicBarrier), "Thread-" + i).start();
        }
    }

    public static Runnable task(CyclicBarrier cyclicBarrier) {
        return () -> {
            System.out.println(Thread.currentThread().getName() + " 开始初始化");
            int r = new Random().nextInt(5) + 1;
            System.out.println(Thread.currentThread().getName() + " 预估初始化时间为: " + r + "秒");
            try {
                Thread.sleep(r * 1000);
                System.out.println(Thread.currentThread().getName() + " 初始化完成, 等待其它线程任务初始化完成");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        };
    }
}
```

该例子会等待其它线程也都初始化完成后, 当CyclicBarrier中的count为0的时候即放行。但是, 我们想等待所有线程都到齐了之后在做一些处理, 那么就需要使用另外的构造方法了。

**样例2: 当所有线程到齐后, 做一些处理工作**

```java
/**
 *      描述:     CyclicBarrier例子使用
 */
public class CyclicBarrierExample02 {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
            System.out.println(Thread.currentThread().getName() + " 调用, 统一执行所有任务");
        });
        for (int i = 0; i < 5; i++) {
            new Thread(task(cyclicBarrier), "Thread-" + i).start();
        }
    }
    // ...
}
```

此例子运行后, 当所有线程都到齐了之后, 由最后一个进入的线程去触发调用我们构造CyclicBarrier传入的Runnable接口方法。

**样例3: 重复使用的CyclicBarrier**

```java

/**
 *      描述:     CyclicBarrier例子使用
 */
public class CyclicBarrierExample03 {
    public static void main(String[] args) {
        for (int i = 0; i < 15; i++) {
            new Thread(task(cyclicBarrier), "Thread-" + i).start();
        }
    }
}
```

这里我们的循环调用改为15次, 前面等到5个任意线程到齐之后就会统一去执行, 然后第二批的5个线程并统一执行依次类推。
但是, 如果不满足我们设定的阈值(假设5)就会一直阻塞。

这里就要引入和CountDownLatch的一些不同点:
  1. CountDownLatch通过countDown()方法来减少计数(我可以调用多次), CyclicBarrier则是通过线程它只有调用await()方法才会减少计数。

  2. CyclicBarrier可以重复使用而CountDownLantern不可以。


```java
/**
 *      描述:     CyclicBarrier例子, 使用单例的线程池数量不满足等待线程数量出现阻塞
 */
public class CyclicBarrierExample04 {

    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(2, () -> {
            System.out.println(Thread.currentThread().getName() + " 调用, 统一执行所有任务");
        });

        CountDownLatch countDownLatch = new CountDownLatch(5);

//        for (int i = 0; i < 2; i++) {
//            new Thread(task1(cyclicBarrier), "Thread-" + i).start();
//        }

        ExecutorService executorService =
                Executors.newSingleThreadExecutor();
        for (int i = 0; i < 2; i++) {
            executorService.execute(task1(cyclicBarrier));
        }

        executorService.shutdown();


//        Thread.sleep(100);
//        new Thread(task2(countDownLatch), "Thread-A").start();
//
//        Thread.sleep(20000);    // 等待时间越长, task2任务阻塞越长,
//        for (int i = 0; i < 5; i++) {
//            countDownLatch.countDown();     // countDown()方法可以在任何你觉得合适的地方去调用执行, 不依赖线程数量
//            System.out.println("当前计数器的值为: " + countDownLatch.getCount());
//        }
    }
}
```

如果我们使用一个单例的线程池, 而我们要等待的线程数有两个。此时线程池中的线程就会被阻塞住无法执行, 而我们使用CountDownLatch则可以直接调用多次来释放。这也得出CyclicBarrier是依赖线程数的。