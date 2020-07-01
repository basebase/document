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

当同学都毕业了, 这本书用不到就可以被回收了。但是有些老实孩子依旧想保存留给自己娃娃, 这个时候这本书就无法被回收了。

###### ThreadLocal共享对象?
每个线程都有自己对象的实例副本, 并且只能由当前线程使用。这就不会出现线程共享同一对象。

但是, 如果真的使用ThreadLocal保存了一个静态变量导致多个线程可以共享此对象, 这就需要使用锁了。因为当多个线程获取到实例并修改会导致线程安全问题, 后面也有例子进行展示。【请注意: 如果把一个共享对象放入到ThreadLocal中, 毫无疑问这是一种没有意义的行为。ThreadLocal没有办法解决这种线程安全问题】

###### ThreadLocal替代Synchronization?
使用ThreadLocal可以让每个线程都持有该对象的一个副本, 由于线程之间的对象实例都是独立的, 因此可以不需要同步, 以便提高程序的性能和可伸缩性能。

但如果说替代Synchronization则完全理解错了ThreadLocal了。它两的工作方式完全是不相同的。当把一个静态对象set到ThreadLocal中, 其它线程同时获取对象并修改, 那么此时如果不使用Synchronization依旧会有线程安全问题。

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




###### 使用ThreadLocal传递参数

通常, 我们的web服务应用可能会使用用户信息, 而我们要将用户信息传递给另外一个方法(比如查询积分), 然后通过这个方法在传递给另外一个方法(比如下订单), 然后该方法又调用另外一个方法(比如取消订单)。等等一系列依赖用户信息的参数方法调用。

如果我们在每个方法中都去传递这个用户信息, 不仅代码冗余而且也不容易维护等等。如下图, 可以看到每次调用方法都要传递Session对象

![ThreadLocal传递参数一](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/ThreadLocal%E4%BC%A0%E9%80%92%E5%8F%82%E6%95%B0%E4%B8%80.png?raw=true)


既然都要使用用户信息, 那我们设置为一个静态的用户信息变量来接收不就行了? 这肯定是不行的, 每一个请求都是不同用户发出来的, 如果用户A下单, 用户B退单, 结果是A用户退单了, 毕竟是一个静态用户对象。所以完全不可行。

既然如此, 那我使用一个Map结构来保存所有用户信息呢, 不就可以解决了, 需要注意的是, 如果使用Map结构来保存用户信息, 由于是多线程环境下肯定会有线程安全问题, 所以我们需要使用synchronized或者ConcurrentHashMap这种线程安全的map来保存用户信息, 无论是加锁还是ConcurrentHashMap都会对性能有所影响, 线程需要等待。如下图, 使用一个线程安全的map结构来存储

![ThreadLocal传递参数二](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/ThreadLocal%E4%BC%A0%E9%80%92%E5%8F%82%E6%95%B0%E4%BA%8C.png?raw=true)


我们想一下, 既然每个线程都需要一个独立的对象进行操作, 我们完全可以使用ThreadLocal来解决啊, 让每个线程都持有一个Session对象, 这样既避免加锁带来的 性能问题, 也解决参数传递带来的冗余操作等。

![ThreadLocal传递参数三](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/ThreadLocal%E4%BC%A0%E9%80%92%E5%8F%82%E6%95%B0%E4%B8%89.png?raw=true)


有了上面的铺垫, 我们可以通过一个具体的例子来实现这样的功能需求了。

```java
/***
 *      描述:     通过ThreadLocal解决传递用户Session信息
 */
public class ThreadLocalSessionTest {

    /***
     *      这里, 我们的threadlocal不在实现任何方法, 但是可以看到下面使用的threadlocal的set方法来设置对象信息。
     */
    public static ThreadLocal<Session> threadLocal = new ThreadLocal();
    public static HashSet<String> hashSet = new HashSet<>();

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 100; i++) {
            final String USER_NAME = "USER-" + i;
            executorService.execute( () -> {
                new Service1().process(USER_NAME);
            });
        }

        executorService.shutdown();
        Thread.sleep(1000);
        if(hashSet.size() > 0)
            throw new IllegalArgumentException("线程不安全啦, 快跑啊!!!");
    }
}

class Service1 {
    public void process(String userName) {
        Session session = new Session(userName);
        ThreadLocalSessionTest.threadLocal.set(session);
        new Service2().process();
    }
}

class Service2 {
    public void process() {
        Session session = ThreadLocalSessionTest.threadLocal.get();
        System.out.println("Service2 Session Info : " + session.name);
        ThreadLocalSessionTest.hashSet.add(session.name);
        new Service3().process();
    }
}

class Service3 {
    public void process() {
        Session session = ThreadLocalSessionTest.threadLocal.get();
        System.out.println("Service3 Session Info : " + session.name);
        ThreadLocalSessionTest.hashSet.remove(session.name);
    }
}

class Session {
    String name ;
    public Session(String name) {
        this.name = name;
    }
}
```

该程序就是, server1调用server2, server2调用server3, 不通过参数的传递, 而是使用threadlocal来完成参数传递, 每一次的请求(即一个线程)只能看到自己的session信息, 而无法修改其它请求的session信息, 即对当前线程所有方法共享了session对象实例, 又避免被其它线程修改。


###### ThreadLocal实例过后的总结
上面两个具体的例子展示了ThreadLocal的一些用法, 第一个例子适用于我们在[ThreadLocal什么时候使用?](#ThreadLocal什么时候使用?)中的第一点和第二点, 而第二个例子适用于我们的第三点。

其实, 简而言之, 我们要判断使用ThreadLocal最初点应该思考是否每个线程都需要独立的对象实例? 如果这个条件都不能成立那么就没有使用的必要了。

**接着, 我们在看看两个实例初始化ThreadLocal方式不同。**  
第一个实例我们在创建ThreadLocal对象会完成initialValue方法的初始化, 而第二个实例中, 我们直接创建ThreadLocal对象就结束了。

咦, 这两种不同创建方式, 有什么不同吗? 那根据什么条件判断使用哪种类型呢?

1. 使用initialValue()方法初始化ThreadLocal对象, 那我们要共享的对象实例是不会变的, 比如我们的工具类, 单例对象, 创建一次, 不会再更新了;

2. 使用set()方法来设置共享对象, 那么被共享的对象是会更新的, 就比如我们传递用户Session对象, 每一次请求的用户都不同;

3. 综上, 如果对象实例仅初始化一次即可则使可以使用initialValue()共享对象, 如果是动态更新共享对象实例使用set()方法。

**使用ThreadLocal带来以下好处**  
  * 达到线程安全;

  * 不需要加锁, 提高执行效率;

  * 更高效利用内存, 节省开销。相比最初每个线程都需要创建一个SimpleDateFormat对象解决线程安全问题, 使用ThreadLocal就可以避免从而节省内存;

  * 避免传参的繁琐;


##### ThreadLocal错误使用

```java

/***
 *      描述:    不正当使用ThreadLocal导致线程安全问题
 */
public class ThreadLocalSimpleDateFormatNoSafe {
    private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    private static ThreadLocal<SimpleDateFormat> threadLocal = new ThreadLocal<SimpleDateFormat>(){
        @Override
        protected SimpleDateFormat initialValue() {
            return simpleDateFormat;
        }
    };

    private static ThreadLocal<SimpleDateFormat> threadLocal2 = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Date date = new Date(1000 * 1);
                String format = null;
                String format2 = null;
                SimpleDateFormat dateFormat = threadLocal.get();
                SimpleDateFormat dateFormat2 = threadLocal2.get();

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                format = dateFormat.format(date);
                format2 = dateFormat2.format(date);

                System.out.println(Thread.currentThread().getName() + " date : " + format);
                System.out.println(Thread.currentThread().getName() + " date2 : " + format2);
            }
        }, "Thread-A").start();

        /// ...
    }
}
```
![threadlocal共享变量错误](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal%E5%85%B1%E4%BA%AB%E5%8F%98%E9%87%8F%E9%94%99%E8%AF%AF.png?raw=true)

上图为运行结果, 可以看到threadLocal使用的是一个静态变量, 静态变量可以被多个线程共享, 所以传入到threadlocal中还是会存在线程安全的。

Threadlocal之所以使得各个线程能够保持各自独立的对象, 并不是通过ThreadLocal的set()方法或者initialValue()方法实现的, 而是通过每个线程中new出来的对象, 并操作这个new出来的对象。

**每个线程线程创建一个新对象, 其实并不是对象的拷贝或者副本[注意: 这里的副本或者拷贝我更多的认为是英文翻译的说法]。**

上面使用静态对象就是最好的例子, 如果ThreadLocal真的会帮助我们创建副本, 就不会出现线程安全问题了。

**[ps: 先看一下ThreadLocal原理的Thread、ThreadLocal和ThreadLocalMap关系图, 再来看我说的下面的话]**

其实, 每个线程都有自己的ThreadLocalMap, 但是我们静态变量就是一个实例, 无论我们的ThreadLocalMap如何存储, 最终都是统一指向这一个对象实例。但是如果是new出来新的对象, 那么每个线程的ThreadLocalMap都有自己的对象实例地址, 这才形成相互不影响。

总结一下:
  1. 如果ThreadLocal.set()的对象本来就是多个线程共享的话, 那么多个线程使用ThreadLocal.get()方法获取到的还是这个共享对象本身, 还是会出现线程安全问题;

  2. ThreadLocal方法并不会为我们创建对象副本, 而是我们自己需要new出一个实例或者使用clone()方法, 这样才是安全正确的使用ThreadLocal;

  3. 既然都是要new对象, 我为什么不使用局部变量? 非要使用ThreadLocal呢?
    * 使用ThreadLocal可以在当前线程内独享, 传递到各个不同的方法中避免参数传递, 而且减少创建对象的开销。


参考[建议都看一下, 第二篇评论区挺精彩]:

[正确理解ThreadLocal](https://www.iteye.com/topic/103804)

[Java并发编程：深入剖析ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html#!comments)


##### ThreadLocal原理

经过上面的学习对ThreadLocal已经有了一个大概的了解以及如何去使用ThreadLocal了。那么, ThreadLocal内部是如何实现的呢? 下面我们就来逐步的分析。


###### Thread、ThreadLocal和ThreadLocalMap关系
在对ThreadLocal源码分析之前呢, 我们先来了解一些基本组件类的关系。Thread, ThreadLocal以及ThreadLoaclMap三者之间的关系。

![threadlocal组件关系图](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal%E7%BB%84%E4%BB%B6%E5%85%B3%E7%B3%BB%E5%9B%BE.png?raw=true)

通过上图可以了解到, 每一个Thread对象都包含一个ThreadLocalMap成员变量, 而ThreadLocalMap中是一个K,V数组, Key就是我们的ThreadLocal, Value是一个Object的对象值。

之所以是一个数组, 这是因为一个线程中可能持有使用N个ThreadLocal对象。

比如, 下面这个例子中就持有两个ThreadLocal变量:
```java

/***
 *      描述:     一个线程中持有多个ThreadLocal变量
 */
public class ThreadLocalMultTest {
    private static ThreadLocal<Test1> test1ThreadLocal = ThreadLocal.withInitial(() -> new Test1());
    private static ThreadLocal<Test2> test2ThreadLocal = ThreadLocal.withInitial(() -> new Test2());
    public static void main(String[] args) {
        ExecutorService executorService =
                Executors.newFixedThreadPool(10);
        for (int i = 0; i < 50; i++) {
            final String NAME = "FINAL-" + i;
            final Integer SCORE = i;
            executorService.execute(() -> {
                /***
                 *      由于这里也不是传递参数什么的, 所以直接get到对象进行set赋值,
                 *      仅仅只是测试方便使用, 不推荐这样写
                 */
                Test1 t1 = test1ThreadLocal.get();
                Test2 t2 = test2ThreadLocal.get();
                t1.setName(NAME);
                t2.setScore(SCORE);

                System.out.println("NAME : " + t1.getName() + " SCORE : " + t2.getScore());
            });
        }
        executorService.shutdown();
    }
}
```


###### ThreadLocal相关方法介绍

1. initialValue()

  * 该方法会返回当前线程对应的"初始值", 这是一个延迟加载的方法, 只有调用get()方法的时候, 才会触发;

  * 如果线程优先调用set()方法, 这种情况下, 不会调用initialValue()方法;

  * initialValue()方法只有在线程第一次调用get()会被触发, 之后就不会被调用了, 但是如果使用了remove()方法后, 在调用get()方法, 则可以再次调用initialValue()方法;

  * 如果不重写initialValue()方法, 该方法会返回null;

2. set(T t)
  * 为当前线程设置一个值;

3. T get()
  * 得到这个线程对应的value, 如果是首次调用get(), 则会调用initialValue()方法获取到对应的副本对象;

4. remove()
  * 删除对应线程的值;


###### ThreadLocal相关方法源码分析


**get()方法的分析**

```java
public T get() {
    // 获取到当前执行线程引用对象
    Thread t = Thread.currentThread();
    // 更具当前线程获取到对应的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);

    /*
        当我们重写initialValue()方法并第一次调用get()方法时,
        ThreadLocalMap返回肯定为null, 首次调用会执行setInitialValue()方法进行一次初始化, 这也就对应我们上面说的延迟加载;
    */
    if (map != null) {
        // 更具当前的ThreadLocal对象获取到对应的Entry对象值
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            // 获取到值, 返回
            T result = (T)e.value;
            return result;
        }
    }

    /**
      这里就是上面说的懒加载, 第一次调用get()时候会初始化值
    */
    return setInitialValue();
}

// 从Thread类中获取到threadLocals成员变量
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}


private T setInitialValue() {
    // 调用我们的initialValue()方法
    T value = initialValue();
    // 获取到当前线程
    Thread t = Thread.currentThread();
    // 更具当前线程获取到对应的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    /*
      如果存在ThreadLocalMap对象, 则把当前ThreadLocal对象作为Key,
      我们要共享的对象设置为Value, 否则就创建ThreadLocalMap对象, 并写入对应信息
    */
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```


![threadlocal-get源码分析1](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal-get%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%901.png?raw=true)

通过Thread-A线程去获取我们ThreadLocal中的副本, 我们记录了当前ThreadLocal的地址信息, 方便后续底层方法对比。

![threadlocal-get源码分析2](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal-get%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%902.png?raw=true)

当我们执行到get()方法的时候, 首先会更具当前线程线程对象引用去获取到对应的ThreadLocalMap对象实例, 如果是第一次调用get那么返回一定是null。所以会进入setInitialValue()方法中。

![threadlocal-get源码分析3](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal-get%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%903.png?raw=true)

进入setInitialValue()方法后, 就会执行initialValue()方法, 也就是我们在创建ThreadLocal类的时候重写的initialValue方法(调用的过程会继续执行get()方法), 如果是首次, 则使用createMap创建ThreadLocalMap否则set更新数据

![threadlocal-get源码分析4](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal-get%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%904.png?raw=true)

执行完setInitialValue()方法后, 在一次去获取ThreadLocalMap不会为null, 之后通过当前ThreadLocal对象去获取ThreadLocalMap中的数组值, 然后返回对象。


**set()方法的分析**

在看完get()方法后, 在来看set()方法, 是不是会感觉轻松很多, 基本就是按照当前线程对象引用去获取ThreadLocalMap, 如果当前线程存在ThreadLocalMap则set值, 否则创建一个ThreadLocalMap对象。

```java
public void set(T value) {
    // 获取当前线程对象引用
    Thread t = Thread.currentThread();
    // 获取当前线程的ThreadLocalMap对象
    ThreadLocalMap map = getMap(t);
    /*
      如果当前线程存在ThreadLocalMap对象使用set设置新值, Key为当
      前ThreadLocal对象, Value为我们set的值。否则创建一个新的ThreadLocalMap对象。
    */
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```


![threadlocal-set源码分析1](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal-set%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%901.png?raw=true)

我们断点到Thread-C线程中, 创建对象后调用ThreadLocal的set方法。

![threadlocal-set源码分析2](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/threadlocal-set%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%902.png?raw=true)

调用时, 会将当前的ThreadLocal最为Key, 传入的值作为Value。


**remove()方法的分析**

```java
public void remove() {
   ThreadLocalMap m = getMap(Thread.currentThread());
   if (m != null)
       m.remove(this);
}
```

这个方法非常简短, 获取当前线程对应的ThreadLocalMap对象引用, 如果线程中存在ThreadLocalMap对象则调用m.remove()方法, 删除当前ThreadLocal的Key。否则就什么都不操作...


**initialValue()方法的分析**

至于initialValue()方法? 先来看看ThreadLocal类中的创建。

```java
protected T initialValue() {
    return null;
}
```

看到没, 就是一个返回null值, 其余什么都没有。所以我们要重写该方法, 否则永远都是一个null值。
