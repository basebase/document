#### Java并发工具3-ThreadLocal

##### ThreadLocal是什么?

###### ThreadLocal简介

首先, 我们来看看Oracle对ThreadLocal的一段描述:

**This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its get or set method) has its own, independently initialized copy of the variable. ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).**

**Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).**

大概意思为:

**ThreadLocal提供线程局部变量。这些变量与普通变量不同, 每个使用该变量的的线程 都有其自己的，独立初始化的变量副本**

**只要线程是活动的并且ThreadLocal 实例是可访问的，则每个线程都对其线程局部变量的副本持有隐式引用。线程消失后，其线程本地实例的所有副本都将进行垃圾回收（除非存在对这些副本的其他引用）**


说了这么多, 到底是个什么意思啊? 说人话好吗?

为了方便理解, 举个例子:

假设线上我们手上只有一本教材, 而现在有30名学生只能用着一本教材上课做笔记, 每位学生都能看到其他学生记录的信息, 甚至去覆盖。

现在, 我们将这一本教材复制30份出来, 每一位同学都在自己拿到的副本上进行修改, 各不干扰。谁都看不见谁的笔记, 也修改不了对方的笔记。

这就是ThreadLocal做的事情, ThreadLocal为每个使用该变量的线程都提供独立的副本变量。所以这就实现每一个线程都可以独立的修改自己的副本对象, 而不会影响其它线程所对应的副本对象。


###### ThreadLocal共享对象?
使用ThreadLocal共享对象? 是认真的吗? 使用ThreadLocal是无法在多个线程之间共享对象的, 除了使用同步之外别无选择。

上面说了, ThreadLocal是创建副本, 每个线程都是独立的副本对象, 不同线程是无法看到其内容的, 如何共享对象呢?


###### ThreadLocal替代Synchronization?
ThreadLocal是解决线程安全的一种方法, 但是它没有解决同步的要求, ThreadLocal是通过向每个线程显示提供对象的副本来消除共享。由于不在共享对象, 因此不需要进行同步,可以提高程序的可伸缩性和性能。


###### ThreadLocal什么时候使用?
1. ThreadLocal非常适合每个线程的Singleton类或者每个线程的上下文信息, 例如事务ID。

2. 可以将任何非线程安全对象包装在ThreadLocal中, 使其成为一个线程安全的。

3. ThreadLocal提供另外一种扩展线程方法, 如果要保留信息获将信息从一个方法调用传递到另外一个方法, 可以使用ThreadLocal进行传递。由于不需要修改任何方法, 因此可以提供极大的灵活性。

如果不是很理解可以参考Java并发编程实战第三章ThredLocal提供的例子, 相信可以更加的理解ThreadLocal。


###### ThreadLocal要点
1. Java的ThreadLocal在JDK1.2引入, 在JDK1.4进行了泛化, 已在ThreadLocal变量上引入类型安全性

2. ThreadLocal可以和Thread空间关联, Thread执行的所有代码都可以访问ThreadLocal变量, 但是两个线程彼此看不到ThreadLocal变量。

3. 每个线程都拥有ThreadLocal变量的副本, 该副本在线程完成或死亡(或异常退出)后才有资格进行垃圾回收, 因此这些ThreadLocal变量没有任何其它的实时引用。

4. Java的ThreadLocal变量通常是类中的私有静态字段, 并在Thread中维护其状态。


虽然ThreadLocal为线程安全开辟了新的道路, 但是不要误解ThreadLocal是Synchronization的替代方案, 它全部取决于设计, 如果对象允许每个线程拥有自己的对象副本, 则可以使用。

概念性可参考:

[ThreadLocal vs Synchronization](https://ranksheet.com/Solutions/kb-Core-Java/1774_ThreadLocal-vs-Synchronization.aspx)

[how-to-use-threadlocal-in-java-benefits](https://javarevisited.blogspot.com/2012/05/how-to-use-threadlocal-in-java-benefits.html#ixzz2Q4g8xqea)


##### ThreadLocal应用实例
上面, 我们已经了解到ThreadLocal特性, 以及应用场景, 下面会使用一些具体例子作为一个展示。

###### ThreadLocal SimpleDateFormat例子
在多线程的环境下使用SimpleDateFormat可能会出现异常, 这是因为SimpleDateFormat不是一个线程安全的类。

在展示ThreadLocal之前, 我们通过一些简单的小例子来发现SimpleDateFormat为什么不安全。

```java
/***
 *      描述:     通过线程池创建N多个线程打印出指定的时间
 */

public class ThreadLocalSimpleDateFormatTest01 {

    public String date(int seconds) {
        Date date = new Date(1000 * seconds);
        SimpleDateFormat dateFormat =
                new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        return dateFormat.format(date);
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            executorService.execute(() -> {
                String date =
                        new ThreadLocalSimpleDateFormatTest01().date(finalI);
                System.out.println(date);
            });
        }
        executorService.shutdown();
    }
}
```

该例子不会引发线程错误问题, 每个线程都创建了各自的SimpleDateFormat对象, 所以不会干扰其它线程, 但是, 我们不可能每提交一个任务就创建一个SimpleDateFormat对象吧? 假设有100w的任务呢? 甚至更多呢? 内存岂不是要爆炸了!?

那好, 既然你不让我创建这么多对象, 那我就只创建一个SimpleDateFormat对象实例, 大家一起共同使用我。

```java
/***
 *      描述:     多个线程共同使用SimpleDateFormat对象, 引发数据错误
 */
public class ThreadLocalSimpleDateFormatTest02 {

    /***
     *  注意这里一定要使用static, 在使用线程池提交任务的时候, 我们每次都是new出ThreadLocalSimpleDateFormatTest02对象的
     *  这还是会导致每个线程都是独立的SimpleDateFormat对象。
     */
    private static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public String date(int seconds) {
        Date date = new Date(1000 * seconds);
        return dateFormat.format(date);
    }

    public static void main(String[] args) throws InterruptedException {

        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            executorService.execute(() -> {
                String date =
                        new ThreadLocalSimpleDateFormatTest02().date(finalI);
                System.out.println(date);
            });

        }

        executorService.shutdown();
    }
}
```

当多个线程共享同一个SimpleDateFormat对象实例的时候, 问题就出现了, 我们打印出来的日期数据竟然出现重复值了, 这明显是线程不安全的。

既然问题出现了, 如何解决呢?

**方法一: 使用synchronized**
```java
/***
 *      描述:     多个线程共同使用SimpleDateFormat对象, 使用synchronized解决数据错误问题
 */
public class ThreadLocalSimpleDateFormatTest03 {
    private static SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    private static HashSet<String> hashSet = new HashSet();

    public String date(int seconds) {
        Date date = new Date(1000 * seconds);

        /***
         *      由于出错的点是在格式化的时候, 所以我们对dateFormat.format进行加锁保护
         */
        String format = null;
        synchronized (ThreadLocalSimpleDateFormatTest03.class) {
            format = dateFormat.format(date);
            if (!hashSet.add(format))
                throw new IllegalArgumentException("出现重复值了...");
        }
        return format;
    }

    public static void main(String[] args) throws InterruptedException {

        ExecutorService executorService =
                Executors.newFixedThreadPool(10);

        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            executorService.execute(() -> {
                String date =
                        new ThreadLocalSimpleDateFormatTest03().date(finalI);
                System.out.println(date);
            });

        }
        executorService.shutdown();
    }
}
```

程序没有抛出异常信息, 已经解决了共享同一个SimpleDateFormat对象实例引发的线程不安全问题, 但是使用synchronized会导致其它线程等待另外一个线程释放锁, 这就会浪费很多时间在等待锁上面了, 加锁虽然可以解决, 但并不是最优的解决方法。


**方法二: 使用ThreadLocal(推荐)**

```java

/***
 *      描述:     多个线程共同使用SimpleDateFormat对象, 使用ThreadLocal解决数据错误问题
 */
public class ThreadLocalSimpleDateFormatTest04 {

    private static ThreadLocal<SimpleDateFormat> threadLocal = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

    private static HashSet<String> hashSet = new HashSet();

    public String date(int seconds) {
        Date date = new Date(1000 * seconds);
        String format = null;
        SimpleDateFormat dateFormat = threadLocal.get();
        System.out.println(dateFormat);
        format = dateFormat.format(date);
        if (!hashSet.add(format))
            throw new IllegalArgumentException("出现重复值了...");
        return format;
    }

    public static void main(String[] args) throws InterruptedException {

        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            executorService.execute(() -> {
                String date =
                        new ThreadLocalSimpleDateFormatTest04().date(finalI);
                System.out.println(date);
            });

        }

        executorService.shutdown();
    }
}
```

使用ThreadLocal后, 每个线程中都持有对SimpleDateFormat对象的副本, 解决了多个线程下使用同一个SimpleDateFormat对象实例带来的线程安全问题, 并且各个线程之间还无需等待对方释放锁, 大大的提升了程序的性能。


至此, SimpleDateFormat可以在多线程环境下安全的运行, 回顾一下最初的一些操作:
  1. 使用线程池来执行线程任务, 但是每次都是创建新的SimpleDateFormat对象, 内存消耗太大。

  2. 既然每个线程都要使用SimpleDateFormat对象, 那么对个线程共享同一个SimpleDateFormat对象实例, 但这引发了数据不安全。

  3. 为了解决共享同一个实例对象引发的线程不安全, 使用synchronized来解决该问题, 但是在高并发的场景下, 这种需要等待锁释放锁的情况极大的消耗资源, 并不是推荐使用的。

  4. 最后利用ThreadLocal来解决线程安全问题并解决了锁带来的性能问题, 同时每个线程内部都有自己SimpleDateFormat对象副本, 不同线程之间无法干扰对方。
