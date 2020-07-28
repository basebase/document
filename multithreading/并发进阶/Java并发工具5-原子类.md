### Java并发工具4-原子类

#### 概述
Atomic(原子)操作一般认为是最小的单位。一段代码如果是原子的, 则表示这段代码在执行过程中要么成功, 要么失败。
原子操作一般都是底层通过CPU的指令来实现。而java.util.concurrent.atomic包下的类, 可以让我们在多线程环境下,
通过一种无锁的原子方式实现线程安全。

比如, 我们常说的i++或者++i在多线程环境下就不是线程安全的, 因为这一段代码包含了三个独立的操作, 在没有使用原子变量之前
我们可以通过加锁才能保证"读->改->写"这三个操作时的"原子性"。


#### Java Atomic原子类纵览

|  类型   | 类  |
|  ----  | ----  |
| Atomic基本类型  | AtomicInteger, AtomicLong, AtomicBoolean|
| Atomic数组类型  | AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray |
| Atomic引用类型  | AtomicReference, AtomicStampedReference, AtomicMarkableReference |
| Atomic升级类型  | AtomicIntegerFieldUpdater, AtomicLongFieldUpdater, AtomicReferenceFieldUpdater |

在JDK8之前只有上面这四种类型, 但是JDK8之后新增了两种累加器。

|  类型   | 类  |
|  ----  | ----  |
| Adder累加器  | LongAdder, DoubleAdder|
| Accumulator累加器  | LongAccumulator, DoubleAccumulator|


#### Java Atomic实例

##### Atomic基本类型例子展示
上面Atomic基本类型有三种, 不过由于他们API几乎都是一样的, 所以这里只需要用一种类型作为展示就可以了。
这里使用AtomicInteger类作为基本的展示。

至于里面的很多方法, 这里不会一一介绍, 自行参考API文档即可。主要做的展示就是在多线程环境下, 原子类一定保证共享变量的安全性。

```java
/***
 *      描述:     Atomic基本类型使用方法, 多线程环境下原子类不加锁依旧保持线程安全, 而非原子类则无法保证
 */
public class AtomicIntegerExample {

    // 创建一个原子变量
    public static AtomicInteger atomicInteger = new AtomicInteger(0);

    // 创建一个非原子变量, 多线程环境下会出现安全问题
    public static volatile Integer basic = 0;


    public static void increment() {
        atomicInteger.getAndIncrement();        // 该方法是获取并自增数据
//        atomicInteger.getAndAdd(10);        // 如果不想自增加1, 可以自定义加想要的值, 还可以是负数
    }

    public static void basicAdd() {
        basic ++;                             //  由于不是原子变量, 所以会出现线程安全问题
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(task(), "Thread-A");
        Thread t2 = new Thread(task(), "Thread-B");

        t1.start();
        t2.start();

        // 等待t1和t2执行完毕
        t1.join();
        t2.join();

        System.out.println("原子变量输出结果为: " + atomicInteger.get());
        System.out.println("非原子变量输出结果为: " + basic);
    }

    public static Runnable task() {
        return () -> {
            for (int i = 0; i < 1000; i++) {
                increment();
                basicAdd();
            }
        };
    }
}
```

输出结果原子变量无论是多少个线程执行, 最终的值都是我们想要得到的结果。而非原子变量又没有被同步代码块保护的话,
则每次运行出来的结果都会不一样。


##### Atomic数组类型例子展示
Atomic数组的使用依旧是以Integer类型举例, 不过Atomic数组中的变量都是原子类型的。无论有多少线程对其修改, 最终结果都是我们想要的值。

```java
/***
 *
 *      描述:     原子数组, 数组中的元素都能保证原子性
 */

public class AtomicArrayExample {

    // 创建一个原子数组对象, 包含1000个元素, 里面元素初始化的值为0
    public static AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(1000);

    public static Runnable addTask() {
        return () -> {
            for (int i = 0; i < atomicIntegerArray.length(); i++) {
                atomicIntegerArray.incrementAndGet(i);       // 以原子方式将索引i处的元素加1
            }
        };
    }

    public static Runnable subTask() {
        return () -> {
            for (int i = 0; i < atomicIntegerArray.length(); i++) {
                atomicIntegerArray.decrementAndGet(i);       // 以原子方式将索引i的元素减1
            }
        };
    }


    public static void main(String[] args) throws InterruptedException {

        Thread[] addThreads = new Thread[100];
        Thread[] subThreads = new Thread[100];
        for (int i = 0; i < 100; i++) {
            addThreads[i] = new Thread(addTask());
            subThreads[i] = new Thread(subTask());
            addThreads[i].start();
            subThreads[i].start();
        }

        for (int i = 0; i < 100; i++) {
            addThreads[i].join();
            subThreads[i].join();
        }

        for (int i = 0; i < atomicIntegerArray.length(); i++) {
            if (atomicIntegerArray.get(i) != 0)         // 获取位置i的当前值
                throw new IllegalArgumentException("原子变量失效, 线程出现安全问题...");

            System.out.println(atomicIntegerArray.get(i));
        }

        System.out.println("安全输出");
    }
}
```

该例子中, 我们创建100个线程对其进行加和减, 无论有多少线程并发去执行, 最后的结果值依旧是我们想要的结果。