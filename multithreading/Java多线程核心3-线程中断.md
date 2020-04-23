#### Java多线程核心3-线程中断

##### 线程中断介绍

##### 前言
当我们开发了一款GUI界面程序会应用到多线程, 比如点击某个杀毒软件的取消按钮来停止查杀病毒, 当我们想停止某个文件下载或者是实现了多线程的任意程序。我们想要中断某一个线程正在执行的任务, 都需要通过一个线程去取消另外一个线程正在执行的任务。Java没有提供一种安全直接的方法来停止某个线程。但是Java提供了中断机制。

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


###### 循环迭代每次try/catch sleep
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
            while (num < 100000) {
                System.out.println("当前值为: " + num);
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                num ++;
            }
        };
    }
}
```

线程会抛出异常, 但可以发现线程没有被中断。还一直在执行, 直到大于while条件或者手动退出。这是为什么呢？

##### 总结

参考:
  1. [处理 InterruptedException](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)
  2. [详细分析 Java 中断机制](https://www.infoq.cn/article/java-interrupt-mechanism)
  3. [Java里一个线程调用了Thread.interrupt()到底意味着什么？](https://www.zhihu.com/question/41048032)
