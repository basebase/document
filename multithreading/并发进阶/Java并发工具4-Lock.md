### Java并发工具4-Lock

#### 概述
对于Java中的锁, 不仅仅是我们之前学习的synchronized关键字。其实juc还提供了Lock接口, 并实现了各种各样的锁。每种锁的特性不同, 可以适用于不同的场景展示出非常高的效率。

由于锁的特性非常多, 故按照特性进行分类, 帮助大家快速的梳理相关知识点。

![java锁脑图](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E9%94%81.png?raw=true)

我们会从下面路径进行学习
  * 乐观锁&悲观锁
  * 可重入锁&非可重入锁
  * 公平锁&非公平锁
  * 共享锁&排它锁
  * 自旋锁&阻塞锁
  * 锁优化


#### Lock还是synchronized?

##### Lock

我们先来看看JDK对Lock的描述:

**Lock implementations provide more extensive locking operations than can be obtained using synchronized methods and statements**

Lock提供比synchronized更广泛的锁操作。

**A lock is a tool for controlling access to a shared resource by multiple threads. Commonly, a lock provides exclusive access to a shared resource**

Lock是一种用于控制多个线程对共享资源独占访问的工具。

**only one thread at a time can acquire the lock and all access to the shared resource requires that the lock be acquired first. However, some locks may allow concurrent access to a shared resource, such as the read lock of a ReadWriteLock.**

一次只有一个线程可以获取到Lock锁, 并且对共享资源的所有访问都需要先获取到锁。
但是, 某些锁可能允许并发访问共享资源, 例如: ReadWriteLock读写锁。

当然, 更多的描述可以自行查阅JDK的文档内容。不过从这里的信息也大概了解到, Lock要做的事情其实是和synchronized一样的, 都是让多个线程获取到锁才能操作共享资源。
只不过, Lock能带来一些灵活的操作, 但是灵活的代价就是维护的一些成本, 后面会说到。


##### synchronized
对Lock有了一个大概的了解后, 那synchronized呢?

鉴于很多地方在学习Lock都会对比synchronized。那就必须要看看使用synchronized会有哪些问题? Lock工具类是否又帮助我们解决了?


使用synchronized保护一段被多个线程访问的代码块, 当一个线程获取到锁后, 其它线程只能等待锁被释放, 而释放锁只有下面两种情况:
  1. 获取锁的线程执行完了同步代码块内容, 正常的释放持有的锁;
  2. 线程执行发生异常, 此时JVM让线程自动释放锁;

假设, 当前获取到锁的线程等待I/O或者长时间sleep被阻塞, 但是锁没有被释放, 其它线程只能安静的等待, 这是非常影响程序的执行效率的。

因此, 就需要一种机制让那些等待的线程别无期限的等待下去了(比如只等待一定时间或者响应中断), 通过Lock就可以实现。


在比如说, 当有多个线程读写文件时, 读操作和写操作会发生冲突, 写操作和写操作也会发生冲突, 但是读操作和读操作不会发生冲突。

如果使用synchronized关键字来实现同步, 就会导致一个问题, 多个线程都是读操作, 当一个线程获取到锁时进行只读操作, 其余线程只能等待而无法进行读操作。

因此, 需要一种机制来使得多个线程都是进行读操作时, 线程之间不会发生冲突, 通过Lock可以实现。

最后, 通过Lock接口可以清楚的知道线程有没有成功获取到锁, 而synchronized则无法办到。


总结一下:
  1. Lock不是Java内置的, synchronized是Java关键字, 因此是内置特性。Lock是一个类, 通过这个类可以实现同步访问;

  2. Lock和synchronized有一点非常不同, 使用synchronized不需要手动释放锁, 当synchronized方法或者synchronized代码块执行完后, 系统会自动让线程释放对锁的占用; 而Lock则必须要手动释放锁, 如果没有释放锁, 可能会出现死锁。

  3. 至于使用Lock还是synchronized, 如果只锁定一个对象还是建议使用synchronized, 使用Lock则需要自己手动释放锁等。


#### Lock实例
上面只是对Lock接口只是大致的一个介绍, 下面我们就开始对Lock接口的API方法进行具体的实例。更加直观的了解Lock的作用。

那么, 我会按照下面这张图的API方法依次开始介绍。

![java锁API方法.png](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E9%94%81API%E6%96%B9%E6%B3%95.png?raw=true)


##### lock()方法
首先来介绍lock()方法, 此方法用于获取锁, 但是如果锁被其它线程持有则进入等待状态, 并且该方法无法被中断, 一旦死锁后就会陷入无限的等待中...

使用Lock接口中的其它获取锁方法, 都必须搭配上unlock()方法, 否则锁不会被释放。从而造成死锁的困境。


使用lock()方法呢, 也有一套模板方法, 如下:

```java
try {
  lock.lock();
  // ...
} finally {
  lock.unlock();
}
```

注意, unlock()方法最好是在finally块中, 即使程序出现异常, 也能正确将锁释放,
避免出现死锁的情况。

当然程序有异常的情况也还是要加入catch块的。但是释放锁必须写在finally中。

下面来看看具体的一个例子

```java

/***
 *      描述:     Lock不会和synchronized一样自动释放锁, 所以需要在finally中释放持有的锁,
 *               保证发生异常时锁能够被释放
 */
public class LockSimpleExample {

    static Lock lock = new ReentrantLock();
    static int count = 0;
    public static void main(String[] args) {
        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            executorService.execute(() -> {
                try {
                    lock.lock();
                    ++ count;
                    System.out.println(Thread.currentThread().getName() +
                            " start job id : " + count);
                    int j = 1 / 0;
                    lock.unlock();

                } finally {
                    // 如果这里不释放锁, 那么就会出现死锁了, 当前线程不释放锁, 其它线程无法获取到锁;
//                    lock.unlock();
                }
            });
        }
        executorService.shutdown();
    }
}
```

可以看到, 我们的unlock()方法在发生异常程序之后, 这就会导致其它线程会一直等待锁。当我们把finally块中代码注释取消就可以正确的释放锁, 即使出现异常也不会受影响、


##### tryLock()方法
该方法尝试获取锁, 如果当前锁没有被其它线程占用, 则获取成功返回true, 否则获取失败返回false。该方法会立即返回, 即使获取不到锁也不会一直等待。

我们也可以给tryLock()方法传递参数, 在指定的时间内尝试获取锁, 如果获取到锁返回true, 否则返回false。

对于tryLock()方法也有对应的模板使用方法, 如下:
```java
if (lock.tryLock() || lock.tryLock(timeout, unit)) {
  try {
    // ...
  } finally {
    lock.unlock();
  }
} else {
  // ...
}
```

当获取到锁的时候, 我们可以处理那些业务逻辑, 当没获取到锁时, 我们又可以处理哪些逻辑。

使用tryLock()方法可以避免出现死锁的问题, 当线程A持有锁1, 线程B持有锁2, 此时线程A想要持有锁2, 使用tryLock()获取失败, 我们就让线程A释放锁1, 此时我们的线程B就可以尝试获取锁1, 如果获取到了则进行相关逻辑处理, 否则释放锁2, 所以当其中一个线程完成逻辑后, 另外一个线程就可以正常的获取到两把锁。

下面来看具体实例:
```java

/***
 *      描述:     tryLock()方法避免死锁的发生, 对比例子LockSimpleExample2.java
 *               本例中解决死锁的问题, tryLock()之所以解决了, 正是因为其获取不到锁不会无休止的等待, 而是会退出等待
 */
public class TryLockSimpleExample {

    static Lock lock1 = new ReentrantLock();
    static Lock lock2 = new ReentrantLock();

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    while (true) {
                        if (lock1.tryLock(1, TimeUnit.SECONDS)) {
                            try {

                                System.out.println(Thread.currentThread().getName() + " 获取到lock1");
                                Thread.sleep(new Random().nextInt(1000));

                                if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                                    try {
                                        System.out.println(Thread.currentThread().getName() + " 获取到lock2");
                                        break;
                                    } finally {
                                        lock2.unlock();
                                        System.out.println(Thread.currentThread().getName() + " 释放lock2");
                                    }
                                } else {
                                    System.out.println(Thread.currentThread().getName() + " 获取lock2失败, 重新获取");
                                }
                            } finally {
                                lock1.unlock();
                                System.out.println(Thread.currentThread().getName() + " 释放lock1");
                            }
                        } else {
                            System.out.println(Thread.currentThread().getName() + " 获取lock1失败, 重新获取");
                        }
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "Thread-A").start();

        new Thread(() -> {
            try {
                while (true) {
                    if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                        try {
                            System.out.println(Thread.currentThread().getName() + " 获取到lock2");
                            Thread.sleep(new Random().nextInt(1000));

                            if (lock1.tryLock(1, TimeUnit.SECONDS)) {
                                try {
                                    System.out.println(Thread.currentThread().getName() + " 获取到lock1");
                                    break;
                                } finally {
                                    lock1.unlock();
                                    System.out.println(Thread.currentThread().getName() + " 释放lock1");
                                }
                            } else {
                                System.out.println(Thread.currentThread().getName() + " 获取lock1失败, 重新获取");
                            }
                        } finally {
                            lock2.unlock();
                            System.out.println(Thread.currentThread().getName() + " 释放lock2");
                        }
                    } else {
                        System.out.println(Thread.currentThread().getName() + " 获取lock2失败, 重新获取");
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Thread-B").start();
    }
}
```


##### lockInterruptibly()方法
lockInterruptibly()方法比较特殊, 当通过这个方法获取锁的时候, 如果线程正在等待锁, 则这个线程可以响应中断。

也就是说当两个线程同时通过lock.lockInterruptibly()方法想获取某个锁时, 假如此时线程A获取到锁, 而线程B只能等待, 那么对线程B调用interrupt()方法能够中断线程B的等待过程。当然, 我们不仅仅可以在等待获取锁的时候中断线程, 在获取到锁之后同样也可以调用interrupt()方法中断在执行的线程。

由于lockInterruptibly()方法会抛出异常, 所以使用lockInterruptibly()方法必须在try/catch块中或者抛出InterruptedException。

下面来看具体实例:

```java
/***
 *
 *      描述:     一个线程获取到锁, 另外一个线程等待锁, 两种情况下中断线程
 */

public class LockInterruptiblySimpleExample {
    static Lock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(task(), "Thread-A");
        Thread t2 = new Thread(task(), "Thread-B");

        t1.start();
        t2.start();

        Thread.sleep(2000);

        /***
         *
         *      当调用线程的interrupt来中断线程, 可能在不同时间节点会有不同的表现, 由于lockInterruptibly()方法本身就会抛出中断异常
         *      所以, 当线程还在等待获取锁的时候就可以被中断。当线程获取到锁依旧可以被中断
         *
         *      当t1线程获取到锁时, 调用interrupt时, 会输出 "休眠期间被中断",
         *      而当t1线程没有获取到锁时, 调用interrupt时, 会输出 "等锁期间被中断"
         *
         *      都可以正常的中断执行的线程
         */

//        t1.interrupt();
        t2.interrupt();

    }

    private static Runnable task() {
        return () -> {
            System.out.println(Thread.currentThread().getName() + " 尝试获取lock");
            try {
                lock.lockInterruptibly();
                try {
                    System.out.println(Thread.currentThread().getName() + " 获取到锁");
                    Thread.sleep(50000);
                } catch (InterruptedException e) {
                    System.out.println(Thread.currentThread().getName() + " 休眠期间被中断");
                } finally {
                    lock.unlock();
                    System.out.println(Thread.currentThread().getName() + " 释放锁");
                }
            } catch (InterruptedException e) {
//                e.printStackTrace();
                System.out.println(Thread.currentThread().getName() + " 等锁期间被中断");
            }
        };
    }
}
```


#### 锁分类

##### 悲观锁与乐观锁
下面会对悲观锁和乐观锁进行以下方面介绍:
  * 悲观锁和乐观锁介绍
  * 悲观锁和乐观锁执行方式图解
  * 悲观锁和乐观锁的例子
  * 乐观锁实现的几种方式
  * 优缺点以及如何选择

###### 什么是悲观锁什么是乐观锁
悲观锁认为自己在使用数据的时候一定有别的线程来修改数据, 因此在获取数据的时候就会先加锁, 确保数据不会被别的线程修改。在Java中synchronized关键字和Lock的实现类都是悲观锁。

对于乐观锁而言, 认为自己使用数据时不会有别的线程修改数据, 所以不会添加锁, 只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新, 则当前线程将自己修改的数据写入。如果数据被其它线程更新, 则更具不同实现方式执行不同的操作(例如报错或者自动重试)。

这里需要注意的是, 乐观锁是通过不加锁的方式进行的, 所以更准确的叫法为
**"乐观并发控制, 英文为(Optimistic Concurrency Control, 可缩写为OCC)"**

在Java中实现乐观锁的方发是CAS算法, 而Java原子类中的递增操作就是通过CAS自旋实现的。


###### 两种锁的执行方式图解

我们通过下面的图来了解一下悲观锁和乐观锁的执行流程。

![悲观锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E6%82%B2%E8%A7%82%E9%94%81.png?raw=true)
![乐观锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E4%B9%90%E8%A7%82%E9%94%81.png?raw=true)

###### 悲观锁和乐观锁的例子

对于上面的描述以及执行流程, 大概清楚了悲观锁和乐观锁, 但还是比较抽象的, 所以下面通过一个具体的例子进行展示。

```java
/***
 *     描述:      展示乐观锁和悲观锁的例子
 */
public class OptimisticAndPessimisticLocking {

    static double amount = 0.0;
    static AtomicInteger count = new AtomicInteger();
    public static void main(String[] args) {
        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 20; i++) {
            executorService.execute(() -> {
                synchronized (OptimisticAndPessimisticLocking.class) {
                    amount += 10.0;
                    System.out.println("amount => " + amount);
                    /***
                     *    在这里, 一个乐观锁对象放入到悲观锁中, 这代表乐观锁是加锁了吗?
                     *      不是的, 这只是乐观锁与加锁操作合作的一个例子, 不能改变, "乐观锁本身不加锁"的事实。
                     */
                    // count.incrementAndGet();
                }

                count.incrementAndGet(); // 自增加1
                System.out.println("count => " + count.get());
            });
        }
        executorService.shutdown();
    }
}
```

通过实例展示, 使用悲观锁基本都是在显示的锁定之后在操作同步资源, 而乐观锁则直接去操作同步资源。那么, 乐观锁为什么能够做到不锁住同步资源也可以正确的实现线程安全呢? 这里就用到了
**CAS** 算法, 在java.util.concurrent包中的原子类都是通过CAS来实现乐观锁。

在这里, 不会展开CAS, 而是会单独写一篇。


###### 乐观锁实现的几种方式
乐观锁的实现方式常见为两种:
  * CAS
  * 版本机制


这里只描述版本机制, 别人不都是版本号机制吗? 是的, 没错, 或许我们在学习的时候都是使用在数据库中添加一个version字段记录版本号来决定当前数据有没有被更新过。

但要注意的是
**使用version仅仅只是一种策略而已, 我们还可以使用时间戳(timestamp), 或者在一些极端情况下没法新增字段使用行本身的状态, 虽然这种策略很糟糕。但也是一种方式。**

举个例子, 假设现在有两人分别为Alice和Bob, 他们分别从账户中取出一定的金额。

![银行转账](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E4%B9%90%E8%A7%82%E9%94%81%E7%89%88%E6%9C%AC1.png?raw=true)

上图中, 我们可以看到Alice认为她可以从她的账户中取出40元, 但是没有意识到Bob刚更新了账户余额。现在账户余额中只剩下20元了。

这个时候, Alice在取出40元的时候, 实际上已经没有富余的钱可以取出了, 如果还让Alice取出来了, 请第一时间告诉我, 我赶紧去取钱。

那要避免这种冲突, 我们一般可以采用悲观锁和乐观锁来实现。


![悲观锁保证](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E4%B9%90%E8%A7%82%E9%94%81%E7%89%88%E6%9C%AC2.png?raw=true)

使用悲观锁, 锁定账户共享资源, 只有获取锁才可以修改账户余额。可以防止另外一个人更改账户。

因此在一个用户释放锁之前, 其他人都无法更改账户的金额。只有在提交事务并释放锁之后, 另外一个用户才可以进行新的update操作。否则会一直被阻塞。

![乐观锁保证](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E4%B9%90%E8%A7%82%E9%94%81%E7%89%88%E6%9C%AC3.png?raw=true)

使用乐观锁允许冲突, 但是在版本更新后进行update时会检测到冲突。

这次, 在account账户中添加了version字段, version列主要作用就是在每次执行update或者delete时都会增加, 并且在update和delete语句的where字句中也使用此列。为此, 我们需要version在执行update或者delete之前发出select并读取当前的值, 否则, 我们将不知道哪个版本值传递给where字句或进行递增。


###### 优缺点以及如何选择

这里更多是如何选择更适合什么场景下的锁

**功能限制**  
与悲观锁对比, 乐观锁适用场景受到更多限制, 无论是CAS还是版本机制。
例如: CAS只能保证单个变量操作的原子性, 当涉及到多个变量时, CAS是无力的, 而synchronized或者Lock则可以通过对整个代码块加锁进行处理。

在比如, select的是表1, 而update的是表2也很难通过比较简单的版本号实现乐观锁。

**竞争激烈程度**  
如果悲观锁和乐观锁都可以用, 就需要考虑竞争的激烈程度:
  * 竞争不激烈(出现冲突概率小)时, 乐观锁更有优势, 使用悲观锁会锁住代码块或数据, 其它线程无法同时访问, 影响并发, 而且加锁和解锁都需要消耗资源。

  * 竞争激烈(出现冲突概率大)时, 悲观锁更体现优势, 因为乐观锁在执行更新时频繁失败, 需要不断重试, 浪费CPU资源。


综上:

悲观锁更适合并发写入多的情况, 适用于临界区持有锁时间比较长的情况, 悲观锁可以避免大量的无用自旋消耗, 考虑下面情况：
  * 临界区有I/O操作
  * 临界区代码复杂或者长时间等待循环等

如果此时使用乐观锁的话, 那么会一直自旋消耗我们的CPU资源, 效率甚至比悲观锁更糟糕。


乐观锁更适合并发读取的场景, 不加锁的特点能够使读操作的性能大幅提升。而在读操作中使用悲观锁无疑是徒增等待的烦恼...



参考  

[Optimistic vs. Pessimistic Locking](https://medium.com/@recepinancc/til-9-optimistic-vs-pessimistic-locking-79a349b76dc8)

[Optimistic vs. Pessimistic locking(stackoverflow)](https://stackoverflow.com/questions/129329/optimistic-vs-pessimistic-locking)

[【BAT面试题系列】面试官：你了解乐观锁和悲观锁吗？](https://www.cnblogs.com/kismetv/p/10787228.html)

[面试必备之乐观锁与悲观锁](https://juejin.im/post/5b4977ae5188251b146b2fc8)


##### 可重入锁与非可重入锁

下面我会对可重入锁与非可重入锁进行以下方面介绍:
  * 可重入锁&非可重入锁概念介绍
  * 可重入锁&非可重入锁执行流程
  * 具体代码展示
  * Java可重入锁&非可重入锁实现原理介绍

###### 什么是可重入锁? 什么是非可重入锁?

可重入锁又被称为
**递归锁**, 是指在同一个线程在外层方法获取锁的时候, 在进入该线程的内层方法会自动获取锁(前提锁对象是同一个对象或者class), 不会因为之前已经获取过还没有释放而阻塞。在Java中ReentrantLock和synchronized都是可重入锁, 可重入锁的一个优点是可以一定程度上避免死锁。

非可重入锁, 在同一个线程中在外层方法获取锁的时候, 执行调用另外一个方法, 那么在调用之前需要将所持有的锁释放掉, 实际上该对象锁已经被当前线程所持有, 且无法释放, 所以会出现死锁。


###### 实例展示

```java

/***
 *      描述: 可重入锁&非可重入锁演示
 */

public class ReentrantLockAndNonReentrantLock {
    private static ReentrantLock lock = new ReentrantLock();
    private static MyLock myLock = new MyLock();

    /***
     *      可重入锁实例, 我们可以看到, 任何一个线程, 只要获取到锁之后, 调用另外一个方法同样是可以进入的。
     */

    public static void eat() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + " 开始吃饭...");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " 吃饭结束...");
            play();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void play() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + " 开始游戏...");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " 结束游戏...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    /****
     *      非可重入锁, 同一个线程想再次进入方法时无法获取到锁, 进行阻塞, 又无法释放锁。导致死锁
     */
    public static void func1() {
        try {
            myLock.lock();
            System.out.println(Thread.currentThread().getName() + "开始执行func1()方法...");
            func2();
            System.out.println(Thread.currentThread().getName() + "func1()方法执行完成...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            myLock.unlock();
        }
    }

    public static void func2() {
        try {
            myLock.lock();
            System.out.println(Thread.currentThread().getName() + "开始执行func2()方法...");
            func2();
            System.out.println(Thread.currentThread().getName() + "func2()方法执行完成...");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            myLock.unlock();
        }
    }



    public static void main(String[] args) {
       new Thread(() -> eat(), "Thread-A").start();
       new Thread(() -> eat(), "Thread-B").start();
       new Thread(() -> eat(), "Thread-C").start();

        new Thread(() -> func1(), "Thread-D").start();
        new Thread(() -> func1(), "Thread-E").start();
    }
}


class MyLock {
    private boolean isLock = false;
    /***
     * 加锁
     * @throws InterruptedException
     */
    public synchronized void lock() throws InterruptedException {
        while (isLock) {
            /**
             *  这里进行阻塞, 模拟必须先释放锁才能在获取锁, 但是当前线程无法释放锁...
             *  这里使用while是防止虚假唤醒调用notify之类的方法, 还必须要把状态变量进行检查
             */
            wait();
        }
        isLock = true;
    }

    /***
     *  释放锁
     */
    public synchronized void unlock() {
        isLock = false;
        notifyAll();
    }
}
```

![可重入锁执行](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81%E6%89%A7%E8%A1%8C.png?raw=true)

上面的例子中, 使用ReentrantLock来锁住资源, 可以通过上图看到线程A调用eat()方法, eat()方法在调用play()方法是在同一个线程栈内的, 因为ReentrantLock是可重入的锁, 所以同一个线程在调用play()方法时可以直接获取到当前的对象锁进行操作。

而如果不是一个可重入锁, 可以看到我们的func1()和func2(), 当前线程在调用func2()方法之前需要释放func1()方法获取的锁对象。实际上该对象锁已经被当前线程所持有, 无法释放, 所以会出现死锁。

###### 可重入锁&非可重入锁执行流程

![可重入锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81.png?raw=true)

![非可重入锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E9%9D%9E%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81.png?raw=true)


###### Java可重入锁&非可重入锁实现原理介绍

之前我们说过ReentrantLock和synchronized都是可重入锁, 那么我们通过可重入锁ReentrantLock以及非可重入锁ThreadPoolExecutor下的Worker类的源码对比分析一下为什么非可重入锁在重复调用同步资源时会出现死锁。

首先ReentrantLock和NonReentrantLock都继承了父类AQS, 其父类AQS中维护了一个同步状态state来计数重入次数, state初始值为0。

当线程尝试获取锁时, 可重入锁先尝试获取并更新state值, state == 0 表示没有其它线程在执行同步代码块, 则把state设置为1, 当前线程开始执行。如果state != 0, 则判断当前线程是否是获取到这个锁的线程, 如果是执行state + 1, 且当前线程可以再次获取锁。

而非可重入锁是直接去获取并尝试更新当前state的值, 如果state != 0的话会导致其获取锁失败, 当前线程阻塞。

释放锁时, 可重入锁同样先获取当前state的值, 在当前线程是持有锁的线程的前提下。如果state - 1 == 0, 则表示当前线程所有重复获取锁的操作都已经执行完毕, 然后该线程才会真正的释放锁。而非可重入锁是在确定当前线程是持有锁的线程后, 直接将state设置为0, 将锁释放。


对于ReentrantLock如何获取state的次数, 我们可以通过getHoldCount()方法获取
```java
private static ReentrantLock lock = new ReentrantLock();
lock.getHoldCount();
```
加锁会增加, 释放锁会减少。

![可重入锁&非可重入锁源码](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81&%E9%9D%9E%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81%E6%BA%90%E7%A0%81.png?raw=true)

参考:

[可重入锁与自旋锁](https://kanonjz.github.io/2017/12/27/reentrantlock-and-spinlock/)

[可重入锁和不可重入锁，递归锁和非递归锁](https://www.cnblogs.com/edison20161121/p/10293156.html)


##### 公平锁与非公平锁
下面会对公平锁和非公平锁进行以下方面介绍:
  * 公平锁和非公平锁概念介绍
  * 公平锁和非公平锁图解流程
  * 公平锁和非公平锁例子展示
  * 公平锁和非公平锁源码简述
  * 优缺点介绍


###### 公平锁和非公平锁概念介绍
其实就是和字面上的意思一样, 在我们日常生活中我们可以按照排队的顺序出餐(这是公平的), 当然如果说我一定要在你之前出餐也是可以的。(非公平的)

而我们的公平锁也是一样, 当有多个线程想要获取锁时, 线程会直接进入队列中排队, 队列中的第一个线程才能获得锁。公平锁的优点就是等待锁的线程不会饿死。
缺点就是整体吞吐效率相对非公平锁要低, 等待队列中除第一个线程外所有线程都会阻塞, CPU唤醒阻塞线程的开销比非公平锁要大。


而非公平锁是多个线程想要获取锁时, 获取不到才会进入等待队列(队尾)中去。但是, 如果此时锁刚好可用, 那么这个线程可以无需阻塞直接获取到锁,
**所以非公平锁有可能出现后申请锁的线程先获取到锁的场景**。非公平锁的优点是可以减少唤醒线程的开销, 整体吞吐效率提高, 因为线程有
一定几率不阻塞直接获取到锁, CPU不需要唤醒所有线程。缺点是处于等待队列中的线程可能会饿死, 或者等很久才会获取到锁。


###### 公平锁和非公平锁图解流程

上面使用文字描述了公平锁和非公平锁, 或许过于抽象, 通过下图例子进行一个介绍或许能更加深刻。

![公平锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%85%AC%E5%B9%B3%E9%94%81.png?raw=true)

上图所示, 假设有一口水井, 有管理员看守, 管理员有一把锁, 只有拿到锁的人才能够打水, 打完水后要把锁还给管理员。
每个过来打水的人都要管理员的允许并拿到锁之后才能去打水, 如果前面有人正在打水, 那么这个想要打水的人就必须排队。
管理员会查看下一个要打水的人是不是队伍里排最前面的人, 如果是的话, 才会给你锁让你去打水; 如果不是排第一的人, 就必须去队尾排队, 这就是公平锁。

![非公平锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81.png?raw=true)

对于非公平锁, 管理员对打水的人没有要求。即使等待队伍里有排队等待的人,
但如果上一个人刚打完水把锁还给管理员并且管理员还没有允许等待队伍里下一个人去打水时,刚好来了一个插队的人,
这个插队的人是可以直接从管理员手里拿到锁去打水, 不需要排队, 原本排队等待的人只能继续等待。


###### 公平锁和非公平锁实例
还是以我们的ReentrantLock为例进行展示。在我们创建ReentrantLock实例的时候可以传入一个布尔值的参数, true代表公平锁, false为非公平锁。


```java
/***
 *      描述:     公平锁&非公平锁展示
 */
public class FairAndNonFairLockExample {
//    公平锁
//    private static Lock lock = new ReentrantLock(true);
    // 非公平锁
    private static Lock lock = new ReentrantLock(false);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            new Thread(task(), "Thread-" + i).start();

            // 按顺序启动线程, 避免在启动的过程中造成优先问题
            // 现在线程的启动执行顺序一定是Thread-0到Thread-9
            Thread.sleep(100);
        }
    }

    private static Runnable task() {
        return () -> {
            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName() + "获取到锁...");
                System.out.println(Thread.currentThread().getName() + " 开始打水...");
                long ms = new Random().nextInt(5);
                System.out.println(Thread.currentThread().getName() + " 需要 " + ms + "秒打水完成");
                Thread.sleep(ms * 1000);
                System.out.println(Thread.currentThread().getName() + " 打水完成, 释放锁...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

            /***
             *  或许打完一次水, 还想在继续打一次水
             */

            try {
                lock.lock();
                System.out.println(Thread.currentThread().getName() + "获取到锁...");
                System.out.println(Thread.currentThread().getName() + " 开始打水...");
                long ms = new Random().nextInt(5);
                System.out.println(Thread.currentThread().getName() + " 需要 " + ms + "秒打水完成 ");
                Thread.sleep(ms * 1000);
                System.out.println(Thread.currentThread().getName() + " 打水完成, 释放锁...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        };
    }
}
```

上面的例子中输出结果
  * 公平锁
    * 首先Thread-0获取到锁, 然后执行, 但是期间Thread-1到Thread-9的线程
    都被加入到等待队列中了, 所以, Thread-0想在一次打水的话会加入到队列的队尾中。此时, 按照我们的构想, 队列应该为  
    Thread-1 -> Thread-2 -> Thread-3 -> Thread-4 -> Thread-5 -> Thread-6 -> Thread-7 -> Thread-8 -> Thread-9 -> Thread-0  
    所以, 我们打印输出的结果顺序一定是:  
    Thread-0 -> Thread-1 -> Thread-2 -> Thread-3 -> Thread-4 -> Thread-5 -> Thread-6 -> Thread-7 -> Thread-8 -> Thread-9  
    ...  
    Thread-0 -> Thread-1 -> Thread-2 -> Thread-3 -> Thread-4 ->
    Thread-5 -> Thread-6 -> Thread-7 -> Thread-8 -> Thread-9

  * 非公平锁
    * 如果是非公平锁的话, 第一个线程Thread-0仍然是先获取到锁, 但是在执行过程中Thread-1到Thread-9都进入等待队列中排队去了....  
    此时, 我们第一次打水完成后, 线程Thread-0释放锁, 但是请注意, 当前线程Thread-0还没有进入阻塞状态, 而我们要唤醒等待队列中的线程是需要时间的, 所以线程Thread-0可能会再一次获取到锁, 从而在打水一次。  
    所以输出结果会出现:  
    Thread-0 -> Thread-0 -> Thread-1 -> Thread-1 -> ... Thread-9

但是, 需要注意的是
```java
tryLock()
```
方法, 它是非公平的方式获取锁, 也就是说无论是在构造参数中创建锁为公平锁也不会按照公平锁的方式进行。

###### 公平锁和非公平锁源码简述

这里, 主要就是简单的介绍一下ReentrantLock类的公平锁和非公平锁的实现不同点, 并非是实现的源码分析。

当我们创建ReentrantLock类, 它的构造方法中有FairSync和NonfairSync两个子类, 它两继承自Sync类, 而Sync继承AQS。

ReentrantLock默认使用非公平锁, 通过上面的学习我们也可以指定为公平锁。

```java
public ReentrantLock() {
     sync = new NonfairSync();
 }

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

下面两张图展示公平锁与非公平锁的加锁方法源码:
![公平锁源码](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%85%AC%E5%B9%B3%E9%94%81%E6%BA%90%E7%A0%81.png?raw=true)
![非公平锁源码](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81%E6%BA%90%E7%A0%81.png?raw=true)

通过图中的源码对比, 可以很明显的看出公平锁与非公平锁的lock()方法唯一的区别在于公平锁在获取同步状态时多了一个限制条件: hasQueuedPredecessors()

![hasQueuedPredecessors方法](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%AD%89%E5%BE%85%E9%98%9F%E5%88%97.png?raw=true)

进入hasQueuedPredecessors()方法, 可以看到该方法主要做一件事情: 判断当前线程是否位于同步队列中的第一个。如果是则返回true, 否则返回false。

综上, 公平锁是通过同步队列来实现多个线程按照申请锁的顺序来获取锁, 从而实现公平的特性。非公平锁加锁时不考虑排队等待问题, 直接尝试获取锁, 所以存在后申请却先获取到锁的情况。

###### 优缺点介绍

|  锁   | 优点  | 缺点  |
|  ----  | ----  | ----  |
| 公平锁  | 线程都有执行的机会, 不会饿死 | 吞吐率低,开销大  |
| 非公平锁  | 吞吐率高 |  线程存在饿死的可能 |


##### 排他锁与共享锁(读写锁)

下面会对排他锁和共享锁进行以下方面介绍
  * 排它锁和共享锁概念介绍
  * 排它锁和共享锁图解流程
  * 排它锁和共享锁例子
  * 读写锁插队策略


###### 排它锁和共享锁概念介绍
对于排他锁, 指的是该锁一次能被一个线程持有。如果线程T对数据A加上排它锁后, 则其它线程不能在对A加任何类型的锁。获得排它锁的线程能读取数据也能修改数据(听起来和我们的悲观锁很像吧)。JDK中的synchronized就是一个排它锁。

共享锁是指该锁可能被多个线程持有, 如果线程T对数据A加上共享锁后, 则其它线程只能对A加共享锁, 不能加排它锁。获得共享锁的线程只能读数据, 不能修改数据。

###### 排它锁和共享锁图解流程

![共享锁和排它锁](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%85%B1%E4%BA%AB%E9%94%81%E5%92%8C%E6%8E%92%E5%AE%83%E9%94%81.png?raw=true)

通过上图可以看到, 当有一个线程要对一个共享资源加锁的时候, 如果要对资源进行写的时候, 那么就加排它锁, 任何线程都不能读取, 但是, 当线程都是只读状态不会对资源做出修改这个时候, 可以多个线程一起去读取里面的内容。

我们发现, 当一个线程在写的时候, 其它线程不能读。当一个或多个线程在读, 其它线程就不能写, 只有读的时候可以有多个线程一起。否则都是形成互斥关系。

读写锁的规则如下:
  * 多个线程申请读锁, 都可以申请到
  * 如果有一个线程已经占用读锁, 此时其它线程如果要申请写锁, 则申请写锁的线程会一直等待释放读锁
  * 如果一个线程占用了写锁, 则此时其它线程如果申请写锁或者读锁, 则申请的线程会一直等待释放写锁。

**所以形成: 要么多读, 要么一写。**


###### 排它锁和共享锁例子
我们通过ReentrantReadWriteLock类, 来实现一个具体的读写例子。

这里, 我们要注意, 读写锁是
**一把锁**, 它不是两把锁, 别看我创建出两个对象。

当我们使用WriteLock的时候, ReadLock肯定是不能用的。当我们用ReadLock则WriteLock也是无法使用的。只有线程都是ReadLock才可以一起获取。
这里可以想成当多个线程进来的时候, 他们是没有办法既获取到读锁又获取都写锁。


而且, 很多人会有疑问, 为什么还需要读锁? 读数据还需要加锁吗? 我又不修改数据, 没错, 但是要考虑如果在读取数据的过程中数据进行了更新, 那么你读取出来的数据还是对的吗? 可以对比例子ReadWriteLockExample2.java就会发现其中最为致命的问题。
当运行ReadWriteLockExample2.java的时候, 你就会觉得ReentrantReadWriteLock确实是一把锁, 否则数据会乱套。

```java
/***
 *
 *      描述:     共享锁和排它锁使用例子
 *                  这里, 我们的读锁就是共享锁。写锁就是排它锁
 *
 *               对比参考例子ReadWriteLockExample2.java
 */
public class ReadWriteLockExample {

    private static ReentrantReadWriteLock reentrantReadWriteLock =
            new ReentrantReadWriteLock();
    // 读锁
    private static ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();

    // 写锁
    private static ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();

    private static int amount = 0;

    public static void main(String[] args) {
        new Thread(writeTask(), "Thread-A").start();
        new Thread(readTask(), "Thread-B").start();
        new Thread(readTask(), "Thread-C").start();
        new Thread(readTask(), "Thread-D").start();
        new Thread(writeTask(), "Thread-E").start();
        new Thread(readTask(), "Thread-F").start();
        new Thread(writeTask(), "Thread-G").start();
        new Thread(readTask(), "Thread-H").start();
        new Thread(readTask(), "Thread-I").start();
    }

    private static Runnable readTask() {
        return () -> {
            readLock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 获取到读锁");
                int random = new Random().nextInt(4) + 1;
                System.out.println(Thread.currentThread().getName() + " 读取数据, 需要" + random + " 秒");
                Thread.sleep(random * 1000);
                System.out.println(Thread.currentThread().getName() + " 读取数据值为: " + amount);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readLock.unlock();
            }
        };
    }

    private static Runnable writeTask() {
        return () -> {
            writeLock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + " 获取到写锁");
                int random = new Random().nextInt(4) + 1;
                System.out.println(Thread.currentThread().getName() + " 更新数据, 需要" + random + " 秒");
                Thread.sleep(random * 1000);
                amount += 10;
                System.out.println(Thread.currentThread().getName() + " 更新数据完成...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                writeLock.unlock();
            }
        };
    }

}
```

###### 读写锁插队策略

既然说到了插队, 我们就会想起了公平锁和非公平锁的策略。对于读写锁也有公平和非公平的策略。

对于公平而言, 那么别想插队了, 老老实实的在等待队列中排队, 到你的时候, 在来执行。所以, 这里我们重点要说的是非公平的下的策略。


假设现在有如下场景:  
现在有线程A和线程B正在读取数据, 线程C想要写入, 但是拿不到锁, 于是进入等待队列中。此时来了一个线程D要进行读操作。

那么, 是直接让线程D读取呢? 还是加入等待队列中呢?

对于读写锁, 此时有两种策略。

策略1:  
读操作可以插队
  * 优点: 效率高
  * 缺点: 容易造成写操作线程饿死

![读写锁插队策略1](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E8%AF%BB%E5%86%99%E9%94%81%E6%8F%92%E9%98%9F%E7%AD%96%E7%95%A51.png?raw=true)

我们通过上图可查看, 如果此时线程D插入执行读操作, 而我们的线程C会继续等待, 如果此时又来了N多获取读锁的线程, 那么想要获取写锁的线程C会一直等待, 直到读操作全部结束, 这就会造成饥饿。


策略2:  
读操作不允许插队
  * 优点: 读写线程有执行机会, 不会饿死。
  * 缺点: 吞吐量下降

![读写锁插队策略2](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E8%AF%BB%E5%86%99%E9%94%81%E6%8F%92%E9%98%9F%E7%AD%96%E7%95%A52.png?raw=true)

透过上图, 可以看到, 新加入的线程, 都到等待队列中排队去。


对于策略2, 如果当前执行的线程获取的读锁, 而等待队列的(队首)节点也是一个获取读锁的线程, 此时可以执行获取读锁执行, 但如果队列的(队首)节点是一个要获取写锁的线程后面才是一个获取读锁的线程, 此时是无法插队执行的。


对于上面两种策略, 我们ReentrantReadWriteLock使用的是策略2。
