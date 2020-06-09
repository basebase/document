#### Java多线程核心4-线程生命周期

##### Java多线程状态(理论)
当我们在创建一个线程, 启动一个线程并执行一个线程, 那么, 这个线程究竟经过了哪些状态的转变？
线程又有多少种状态呢?

那么, 我们首先来看一下, 线程究竟有多少种状态。这里, 我直接将Thread类的内部类信息复制出来。

```java
// Thread.Java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

Thread类定义一个枚举类来描述线程状态信息, 线程一共有6种状态。

那么线程这6种状态如何去理解呢?

1. 初始状态(NEW)
  - 新创建一个线程对象, 但还没有调用start()方法。此时的线程就已经进入了初始状态

2. 运行状态(RUNNABLE)
这里会比较复杂一点, Java线程将就绪(ready)和运行中(running)两种状态笼统的称为"运行"。
  - 就绪状态(就绪状态表示可以执行, 但是还没轮到你, 你就先在这等着执行 )
  - 运行中状态

3. 阻塞状态(BLOCKED)
  - 在进入synchronized关键字修饰的方法或者代码块时(锁已经被其它线程获取)的时候进入此状态

4. 等待状态(WAITING)
  - 该状态下线程不会被执行, 它们需要被显示的唤醒, 否则会处于无限期的等待状态

5. 超时等待(TIMED_WAITING)
  - 该状态和WAITING状态很像, 只不过它有一个时间期限, 如果没有被显示的唤醒, 只需要等到设置时间超过后会被自动唤醒。

6. 终止(TERMINATED)
  - 当线程的run()方法完成时, 当然也包括异常终止。


附上一张Java线程声明周期图
![Java线程生命周期](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.png?raw=true)



##### Java多线程状态(实战)

在了解上面线程的声明周期以及如何转化状态后, 我们通过两个例子来实践看看是不是和上面说的一样。


首先, 我们先把最简单的3种状态优先展示即:  NEW -> RUNNABLE -> TERMINATED状态

```java
/***
 *      描述:     展示线程    NEW -> RUNNABLE -> TERMINATED状态
 */
public class NewRunnableTerminated {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(getRunnable());

        // 在还没有调用start()方法的时候, 线程状态一定是NEW
        System.out.println("还没调用start()方法时候的状态: " + t.getState());

        /***
         *     注意:  这里可能在调用start()方法后, 线程可能会立即获取到CPU执行
         *           如果输出在线程中间, 多运行几遍
         *
         *           不过, 即使在等待被执行或者已在执行中其状态都是RUNNABLE, 不用过于纠结。
         */
        // 调用start()方法, 可能当前线程不会被立即执行, 但是这个时候也会是RUNNABLE而不会出现我们文章说的ready和running两种状态
        t.start();
        System.out.println("调用start()线程还未执行的状态: " + t.getState());

        // 休眠一些时间, 让线程t有机会执行, 执行过程也是RUNNABLE状态
        Thread.sleep(10);

        System.out.println("线程执行时候的状态: " + t.getState());

        // 我们等待线程t运行完成后, 在输出线程状态
        Thread.sleep(100);
        System.out.println("线程结束状态: " + t.getState());

    }

    public static Runnable getRunnable() {
        return () -> {
            for (int i = 0; i < 1000; i ++) {
                System.out.println("当前值为: " + i + " 当前线程 " + Thread.currentThread().getName() + " 状态: " + Thread.currentThread().getState());
            }

        };
    }
}
```


下面这个例子主要展示Blocked, Waiting, TimedWaiting三种状态

```java

/***
 *      展示:     展示 Blocked, Waiting, TimedWaiting三种状态
 */
public class BlockedWaitingTimedWaiting {

    public static void main(String[] args) throws InterruptedException {
        BlockedWaitingTimedWaiting blockedWaitingTimedWaiting = new BlockedWaitingTimedWaiting();
        Runnable runnable = blockedWaitingTimedWaiting.getRunnable();


        /***
         *      需要创建两个线程, 否则无法模拟出BLOCKED的状态, 毕竟只有一个线程的话永远都能获取到锁
         *      两个线程始终有一个线程需要等待获取到锁从而进入BLOCKED状态。
         */

        Thread t1 = new Thread(runnable);
        t1.start();

        Thread t2 = new Thread(runnable);
        t2.start();

        // 当线程t1执行时, 进入休眠状态, 但是有时间限定, 所以, 此时输出线程t1的状态是TIMED_WAITING
        System.out.println(t1.getState());
        // 而线程t2没有获取到锁(此时锁还在线程t1上面), 所以会进入BLOCKED状态
        System.out.println(t2.getState());

        Thread.sleep(3000);
        // 当我们执行wait()方法时, 线程就会进入WAITING状态, 等待被唤醒。
        System.out.println(t1.getState());
    }

    public Runnable getRunnable() {
        return () -> {

            synchronized (this) {
                try {

                    Thread.sleep(2000);

                    // 在等待睡眠结束时, 我们调用wait()方法, 线程会进入WAITING状态
                    wait();

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
    }
}
```
