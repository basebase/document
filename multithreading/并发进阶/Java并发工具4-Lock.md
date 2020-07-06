#### Java并发工具4-Lock

##### 概述
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


##### Lock还是synchronized?

###### Lock

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


###### synchronized
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


##### Lock实例
上面只是对Lock接口只是大致的一个介绍, 下面我们就开始对Lock接口的API方法进行具体的实例。更加直观的了解Lock的作用。

那么, 我会按照下面这张图的API方法依次开始介绍。

![java锁API方法.png](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E9%94%81API%E6%96%B9%E6%B3%95.png?raw=true)


###### lock()方法
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


###### tryLock()方法
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


###### lockInterruptibly()方法
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
