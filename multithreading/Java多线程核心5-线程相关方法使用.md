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
  * 唤醒一个等待的线程(如果有多个线程需要被唤醒, 那么选择唤醒哪一个线程是不确定的)


有了上面对两个方法的宏观认识之后, 我们通过具体的代码实现看看wait()方法和notify()的使用方法

###### wait/notify配合使用入门例子
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
