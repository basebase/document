### Java并发工具6-CAS

#### 概述
CAS的全拼为(compare and swap)中文一般称为比较并交换, 它是原子操作的一种。可用于在多线程中实现不被打断的数据交互操作, 从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题。

该操作通过将内存的值与指定数据进行比较, 当数值一致时将内存中的数据值替换为新的值。

通常来说, 当有多个线程去修改一个值的时候, 只有一个线程可以修改程序, 其余的线程都会失败。


#### CAS实现原理
之前学习过的锁章节中, 有悲观锁和乐观锁, 而我们的乐观锁就是基于CAS实现的。但是当时并没有具体介绍什么是CAS。
CAS又是如何更新我们的值呢?

CAS执行有三步:
  * 从内存地址X读取值V(之后要更新的值)
  * 预期值A(上一次从内存中读取的值)
  * 新值B(写入到V上的值)

当从理论上来看或许会有点晕乎, 什么V什么A和B, 都是些啥?

这里先用伪代码来分解CAS的流程, 之后会通过Java来模拟一个CAS执行流程。

假设我们有个整形变量V为10, 当前有N个线程想要递增该值并用于其它操作。让我们来看看CAS是如何执行的:

1) 现在有线程1和线程2都想要递增V, 它们读取值之后将其增加到11
```java
V = 10, A = 0, B = 0
```

2) 假设线程1首先执行, V将于线程1读取到的值进行比较
```java
V = 10, A = 10, B = 11

if V == A
  V = B
else
  operation failed
  return V
```

很显然, V的值将被覆盖为11, 操作成功。

3) 此时线程2到来并尝试与线程1相同的操作

```java
V = 11, A = 10, B = 11
if V == A
  V = B
else
  operation failed
  return V
```
这种情况下, A的值不等V, 因此不替换值。并返回V当前的值, 即11。线程2此时再次使用获取到的值进行重试

4) 线程2再次执行
```java
V = 11，A = 11，B = 12
```
这一次的A的值是和V相等的。增量值为12, 返回线程2。



当我们使用 
```java
if V == A
```
这个操作就是CAS的第一步和第二步。从内存地址读取值V和我们的预期值A进行比较, 如果一致就会执行第三步更新变量值V
```java
V = B
```

上面就是一个CAS的执行流程, 简而言之, 当多个线程尝试使用CAS同时更新一个变量时, 其中一个线程更新成功其余线程都将失败。
其余更新失败的线程可以进行重试又或者什么都不处理。

![cas更新数据](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/cas-01.png?raw=true)


#### 模拟CAS

前面已经基本上了解了CAS的原理, 下面我们就动手实现一个自己的CAS算法。
其实按照上面的CAS三个步骤实现一个CAS并非难事。

现在, 我们就要实现实现一个数值累加器的程序, 在实现之前, 我们使用synchronized来完成原子和可见性。

```java
/***
 *
 *      描述:     非CAS累加变量, 基于synchronized
 */

public class Counter1 {

    private int value = 0;

    /***
     *
     * 使用synchronized将导致过多的上下文切换, 性能消耗大。
     */

    public synchronized int getValue() {
        return value;
    }

    public synchronized int increment() {
        return ++ value;
    }
}
```

上面的程序可以实现我们的数值累加但是并非使用CAS算法实现。我们需要将其改进。

创建一个CAS的类, 实现CAS算法核心逻辑。
```java
/***
 *      描述:     模拟CAS执行
 */
public class EmulatedCAS {

    private int value = 0;

    public synchronized int getValue() {
        return value;
    }

    /***
     * 此方法就是一个CAS的实现算法, 判断传入的预期值是否和当前的value值一样, 如果一样则更新value并返回上一次内存结果值
     * 这里需要使用synchronized来完成原子和可见性的操作, 否则多个线程执行会出现问题。
     *
     * 我们假设底层是非synchronized实现的即可。
     * @param expectedValue
     * @param newValue
     * @return
     */
    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int readValue = value;
        if (expectedValue == readValue) {
            value = newValue;
        }

        return readValue;
    }
}
```

当我们调用compareAndSwap()方法时, 就需要传入预期值A和更新的值B了。如果有一个线程更新成功, 其它线程读取的还是上一次的旧值, 但是在处理这块逻辑则是由用户自行编写的, 可以循环直到正确。

我们CAS类编写好后, 在重新编写一个累加的程序。

```java
/***
 *
 *      描述:     使用CAS算法累加变量
 */
public class Counter2 {

    private EmulatedCAS value = new EmulatedCAS();

    public int getValue() {
        return value.getValue();
    }

    public int increment() {
        int readValue = getValue();

        /***
         *      这里我们就是使用自己模拟的CAS算法实现的, 当我们的值不同的时候就循环直到正确为止
         */

        while (value.compareAndSwap(readValue, readValue + 1) != readValue) {
            readValue = value.getValue();
        }

        return ++ readValue;
    }

    public static void main(String[] args) throws InterruptedException {
        Counter2 c = new Counter2();

        Thread[] t = new Thread[100000];
        for (int i = 0; i < 100000; i++) {
            t[i] = new Thread(() -> c.increment(), "Thread-" + (i + 1));
        }

        // ...
    }
}
```

这里实现的就是我们自己编写的CAS算法, 最终的结果集也是正确的。


#### CAS存在的问题

虽然CAS提供了无锁的算法, 但是也有自身的缺点:
1. ABA问题:  
  ABA存在于无锁算法中常见的一个问题, 可表述为:
    * 进程P1读取一个数值A;
    * P1被挂起(时间片耗尽, 中断等), 进程P2开始执行;
    * P2修改数值A为数值B, 然后又修改回A;
    * P1被唤醒, 比较后发现数值A没有变化, 程序继续;

2. 循环时间开销大, 自旋CAS如果长时间不成功, 会给CPU带来非常大的执行开销。

3. 只能保证一个共享变量的原子操作, 当对一个共享变量执行操作时, 可以使用循环CAS的方式保证原子操作, 但是对于多个共享变量操作时, CAS就无法保证操作的原子性。



这里, 我提供一个会引起ABA问题的例子。仅供参考:

```java
/***
 *      描述:     ABA问题
 */
public class ABAExample {

    public static AtomicInteger cas = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(task1(0, 1), "Thread-A");
        Thread t2 = new Thread(task2(0, 2), "Thread-B");

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("最终的结果为: " + cas.get());
    }

    public static Runnable task1(int expect, int update) {
        return () -> {
            System.out.println(Thread.currentThread().getName() + " 开始执行程序...");
            int random = new Random().nextInt(6) + 1;

            System.out.println(Thread.currentThread().getName() + " 程序执行需要: " + random + " 秒");
            try {
                Thread.sleep(random * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            while (!cas.compareAndSet(expect, update)) {
            }

            System.out.println(Thread.currentThread().getName() + " 输出结果为: " + cas.get());
        };
    }


    public static Runnable task2(int expect, int update) {
        return () -> {
            System.out.println(Thread.currentThread().getName() + " 开始执行程序...");
            int random = new Random().nextInt(1) + 1;
            System.out.println(Thread.currentThread().getName() + " 程序执行需要: " + random + " 秒");
            try {
                Thread.sleep(random * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            while (!cas.compareAndSet(expect, update)) {

            }

            System.out.println(Thread.currentThread().getName() + " 当前结果为: " + cas.get());

            System.out.println(Thread.currentThread().getName() + " 开始消费金额 ");
            for (int i = 0; i < 2; i++) {
                cas.decrementAndGet();
            }

            System.out.println(Thread.currentThread().getName() + " 消费和的金额为: " + cas.get());
        };
    }
}
```

最终的结果会输出1, 但是我们的线程Thread-B已经修改过数值了, 当线程Thread-A从阻塞中醒来后发现还是原来的值0时其实早就不是之前的值的了。

#### 参考

[比较并交换](https://zh.wikipedia.org/wiki/%E6%AF%94%E8%BE%83%E5%B9%B6%E4%BA%A4%E6%8D%A2)

[Java Compare and Swap Example – CAS Algorithm](https://howtodoinjava.com/java/multi-threading/compare-and-swap-cas-algorithm/)

[Java 101: Java concurrency without the pain, Part 2](https://www.infoworld.com/article/2078848/java-concurrency-java-101-the-next-generation-java-concurrency-without-the-pain-part-2.html?page=3)

[The ABA Problem in Concurrency](https://www.baeldung.com/cs/aba-concurrency)

[非阻塞同步算法与CAS(Compare and Swap)无锁算法](https://www.cnblogs.com/mainz/p/3546347.html)

[并发编程—CAS（Compare And Swap）](https://segmentfault.com/a/1190000015239603)