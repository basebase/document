#### Java多线程核心8-线程安全

##### 什么是线程安全?
在写这个之前, 我会引用一些我觉得还不错的文章或者书籍等, 然后我们来看看每条引用有什么不同之处。

线程安全的定义, 这里我会引用一些比较权威的定义, 最后我自己会试图总结一下线程安全, 每个人的理解都不同, 但是最终的结果一定是一样的, 我也会给出国外一些线程安全讨论的的帖子(个人觉得浏览或者支持高的一定就是好的, 所以给出链接, 自己思考)。

Java并发实战:
  ```text
    当多个线程访问某个类时, 不管运行环境采用何种调度方式或者这些线程将如何
    交替执行, 并且在主调代码中不需要任何额外的同步或者协同,
    这个类都能表现出正确的行为, 那么就称这个类是线程安全的。
  ```

[维基百科](https://zh.wikipedia.org/wiki/%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8):
  ```text
    线程安全是程序设计中的术语, 指某个函数、函数库在多线程环境中被调用时,
    能够正确的处理多个线程之间的共享变量, 使该程序正确完成。
  ```

这两条引用是比较权威的, 这里我做个简单的总结:  
**无论线程如何执行, 如何处理, 线程计算后的结果和我们想要的结果是一样的, 我们可以认为这个线程是安全的, 即所见即所得。**

[Quora上线程安全的讨论](https://www.quora.com/What-does-the-term-thread-safe-mean-in-Java)

[stackoverflow上线程安全的讨论](https://stackoverflow.com/questions/261683/what-is-the-meaning-of-the-term-thread-safe)



##### 线程不安全实例

虽然知道了线程安全的定义, 但是在我们的日常开发中还是会出现一些线程不安全的例子,下面我们会给出一些线程不安全的情况, 分析如何导致的线程不安全, 并且给出解决方案。

###### 多个线程交替运行结果错误
这是一个比较典型的线程不安全的例子, 该类有一个变量value, 多个线程一起执行value++, 可能会导致最终的结果小于我们的预期结果。

```java
/***
 *      描述:     多个线程运行结果出错, 运行后的结果会小于预期结果, 找出问题并解决
 */
public class MultiThreadsError {

    static int value = 0;
    public static void main(String[] args) throws InterruptedException {
        Runnable r = task();
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("最终的结果为: " + value);
    }

    public static Runnable task() {
        return () -> {
            for (int i = 0; i < 10000; i++) {
                value ++;
            }
        };
    }
}
```

上面的例子中, 可以看到每次运行的结果都不相同。有时候是我们预期的值, 有时候缺少值。这是什么原因导致的呢? 我们先来看看下图:

![多线程累加变量](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%B4%AF%E5%8A%A0%E4%B8%80%E4%B8%AA%E5%8F%98%E9%87%8F.png?raw=true)


图片中有两个线程A和B, 虽然递增运算value++看上去是一个单个操作, 但事实上包含了三个独立的操作:
  * 读取value
  * 将value加1
  * 将计算结果写入value

由于运行时间可能将多个线程之间的操作交替执行, 因此这两个线程可能同时执行读操作, 从而使它们得到相同的值, 并都将这个值加1。结果就是, 在不同线程的调用中返回了相同的数值。


上面的例子中, 我们想知道++操作在哪里消失的? 又消失了多少呢?  
下面我们会一点一点的去实现这个功能点, 并知道每个修复点会出现的问题。


**修复1: 我们通过利用一个boolean类型数组记录每次value的值, 如果当前value的值不存在设置为true, 否则就输出异常信息。用来记录在哪个下标位置出现过异常信息。**

为了方便起见, 这里仅贴出重要代码。

```java
// MultiThreadsErrorMark.java
public static Runnable task() {
    return () -> {
        for (int i = 0; i < 10000; i++) {
            /***
             *      既然数据是在这里消失的, 那么每次累加完后就去检查value的值
             *      从而确定是在哪里消失的。
             */
            value ++;
            // 真实累加次数
            realCount.incrementAndGet();

            /***
             *      具体如何实现呢, 我们利用一个数组存入每次累加的值, 如果不存在设置为true
             *      否则我们打印出错误信息
             */

            // 如果该位置已经被标记过了, 则发生了数据错误。
            if (marked[value]) {
                System.out.println("发生了错误: " + value);
                wrongCount.incrementAndGet(); // 出错次数
            }

            /**
             *  当线程1累加完value后当前的value就设置为true, 如果线程2读取并累加的值和线程1一样
             *  那么, 就出现问题
             */
            marked[value] = true;
        }
    };
}
```

这里我们利用了两个原子类记录运行总次数和出错次数。但是最终的运行结果却会发现
我们的“最终输出结果” + "出错次数" != "总运行次数"

为什么会出现该问题? 又是怎么发生的? 如何解决呢?

```java
value ++;
if (marked[value]) {
    System.out.println("发生了错误: " + value);
    wrongCount.incrementAndGet(); // 出错次数
}

marked[value] = true;
```
假设线程1和线程2执行完value++后为3, 线程1判断当前位置没有写入进行赋值
marked[value] = true;但是在还没有完全写入之前, 线程2进入并判断位置3有没有被写入, 由于线程1还没有写入所以线程2依旧会执行marked[value] = true;
这个时候已经发生了冲突, 但是这里的判断却没有感知到错误。


**修复2: 既然上面的问题是多个线程同时执行导致的, 那么我们通过sync来保护这段代码呢?**

```java
public static Runnable task() {
    return () -> {
        for (int i = 0; i < 10000; i++) {
            value ++;
            realCount.incrementAndGet();

            synchronized (lock) {
                if (marked[value]) {
                    System.out.println("发生了错误: " + value);
                    wrongCount.incrementAndGet();
                }
                marked[value] = true;
            }
        }
    };
}
```

这段代码依旧不能对应回去数值, 这是为什么呢?

假设现在的线程1的value进行++后为1, 并且进入了sync代码块, 如果按照我们的想法线程2也是1的话, 就会打印异常信息, 此时线程2进入并进行value++为2, 在次切换回线程1的时候, 此时的value就会为2而不是之前的1了。而线程2进入的时候就会发现value=2的位置已经写入过, 所以要打印异常信息, 本来是没有冲突的数据也变成冲突数据了。


**修复3: 既然在value++的时候还是有问题, 那我们干脆就等待value++完成后统一执行吧?**

```java
public static Runnable task() {
    return () -> {
        for (int i = 0; i < 10000; i++) {
            try {
                cyclicBarrier2.reset();
                cyclicBarrier1.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }

            value ++;

            try {
                cyclicBarrier1.reset();
                cyclicBarrier2.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }

            realCount.incrementAndGet();

            synchronized (lock) {
                if (marked[value]) {
                    System.out.println("发生了错误: " + value);
                    wrongCount.incrementAndGet();
                }
                marked[value] = true;
            }
        }
    };
}
```

上面的代码依旧存在问题, 我们通过CyclicBarrier1等待两个线程都进入for循环后同时去调用value++来触发问题, 但是, 我们说可能会出现value的篡改, 利用CyclicBarrier2等待两个线程进行完value++后才进行后面的sync代码块。

但是, 由于sync的可见性, 线程1和线程2对value的修改, 是可以感知的, 这就导致线程出现每次value的值都相同。进而出现一个错误的异常判断。


**修复4: 既然正确的累加当成了错误, 如果是正确累加的话都是在偶数位置标记为true, 那我们就判断前一位是否为true, 如果是则出现异常**

```java
public static Runnable task() {
    return () -> {
        marked[0] = true;
        for (int i = 0; i < 10000; i++) {
            synchronized (lock) {
                // 新增加前一个位置的判断
                if (marked[value] && marked[value - 1]) {
                    System.out.println("发生了错误: " + value);
                    wrongCount.incrementAndGet();
                }
                marked[value] = true;
            }
        }
    };
}
```

运行上面的程序后, 我们可以得到一个正确的数值了, 即: “最终输出结果” + "出错次数" = "总运行次数"。

这里我们主要对前一个标记为进行判断, 如果value累加是正确的话即线程1和线程2加2次,我们会在2的位置标记true, 4的位置标记true, 以此类推...直到结束。

如果标记位置3为true, 则表示这次累加失败丢失了值。

这里还需要注意的是, 数组的第一个位置(下标为0)比较特殊, 需要手动设置为true, 如果第一次两个线程累加就失败的话, 就会在位置2(下标1)进行标记, 如果是正确的就会在位置3(下标2)进行标记。



###### 死锁问题

死锁的概念其实很简单, 比如现在川普有"懂王"的称号, 奥巴马也想拥有但是川普不给, 奥巴马就去获取"建国"的称号, 而川普呢也想要"建国"的称号, 但是奥巴马不给, 两个人对各自的称号都不放手, 就会一直打嘴炮永不停歇。

简单来说, 当两个以上的运算单元，双方都在等待对方停止运行，以获取系统资源，但是没有一方提前退出时，就称为死锁。


既然知道了死锁, 那可以实现一个死锁吗? 开玩笑, 这还不简单?!

```java


/***
 *      描述:     线程安全, 死锁问题
 *               手动写一个一定会死锁的例子
 */
public class MultiThreadsDeadLockError {
    private static Object lock1 = new Object();
    private static Object lock2 = new Object();

    private static Runnable task1() {
        return () -> {
            synchronized (lock1) {
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 获取到lock1了...");
                System.out.println(Thread.currentThread().getName() + " 尝试获取lock2...");
                synchronized (lock2) {
                    System.out.println(Thread.currentThread().getName() + " 获取到lock2了...");
                }
            }
        };
    }

    private static Runnable task2() {
        return () -> {
            synchronized (lock2) {
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 获取到lock2了...");
                System.out.println(Thread.currentThread().getName() + " 尝试获取lock1...");
                synchronized (lock1) {
                    System.out.println(Thread.currentThread().getName() + " 获取到lock1了...");
                }
            }
        };
    }
}
```

这个程序呢就会一直阻塞下去, 双方都在等待对方释放资源, 可惜没有谁愿意先释放。
对于如何解决这个死锁, 大家可以思考一下如何解决, "notify/notifyAll"。



##### 对象发布与逸出

什么是对象发布?
  + **使对象能够在当前作用域之外的代码中调用。**

例如, 将一个指向该对象的引用保存到其它代码可以访问的地方, 或者在某一个非private的方法中返回该引用, 或者将引用传递到其它类的方法中。

我们将这种可以被外部代码访问类为对象的发布。

那什么是逸出呢?
比如, 我们发布了一个private修饰的对象(例如我们的配置类等), 然而在多个线程中进行了修改会导致最终的配置对象类出现异常, 这种情况称为逸出。


###### 发布并逸出例子
从文字上来看可能比较晦涩, 通过一个简短程序来观察一下, 什么是对象的逸出, 对象逸出的后果有多严重


```java


/***
 *      描述:    对象的发布与逸出
 */
public class MultiThreadsErrorStates {
    public static void main(String[] args) {
        MultiThreadsErrorStatesInner statesInner =
                new MultiThreadsErrorStatesInner();

        Map<String, String> states = statesInner.getStates();

        /***
         *  假设现在有两个线程执行先后顺序我们不确定, 它们就有能力去修改这个不应该被发布的map对象
         *  甚至还可以修改map中存储的对象数据, 以及删除数据。 导致数据不一致等
         */

        new Thread(() -> {
            System.out.println(states.get("K1"));
        }).start();


        new Thread(() -> {
            states.remove("K1");
        }).start();
    }
}

class MultiThreadsErrorStatesInner {
    // 当前设置一个私有变量map作为我们的配置信息项
    private Map<String, String> states ;

    /**
     * 通过构造方法初始化我们的配置项数据
     */
    public MultiThreadsErrorStatesInner() {
        this.states = new HashMap<>();
        states.put("K1", "赛车手");
        states.put("A1", "中门对狙");
        states.put("C4", "炸弹爆炸");
        states.put("F4", "东北F4");
    }

    /**
     * 但是, 我们在一个public修饰的方法中把我们私有变量发布出去了, 可能导致其它类获取到
     */
    public Map<String, String> getStates() {
        return states;
    }
}
```

其实通过这里简短的小例子可以很清除的看到之后会发生哪些问题, 比如配置项被某个线程删除, 配置项被其它线程更新等等。


###### this引用逸出
比如我们在构造函数中, 还未完成初始化对象就进行this复制, 导致对象逸出。

```java
/***
 *      描述:     初始化未完成, 进行this赋值
 */
public class MultiThreadsErrorThis {
    static Point point = null;

    public static void main(String[] args) throws InterruptedException {

        new Thread(() -> {
            try {
                new Point(1, 2);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        /***
         *      我们通过子线程去创建Point对象, 如果在主线程中等待的时间不一致获取到的对象值也不一样
         *      正确数据对象应该是: 在对象不进行更新修改, 我们看到的数据始终一致。
         *
         *      所以, 在初始化未完成的情况进行this赋值会造成this引用逸出。
         */

        // 如果等待50毫秒的话, 可能输出就是1, 0或者其他
//        Thread.sleep(50);

        // 但是如果我们等待时间比较长的话, 输出是1, 2这样的值
        Thread.sleep(120);
        System.out.println(point);
    }
}

class Point {
    private final int x, y;

    Point(int x, int y) throws InterruptedException {
        this.x = x;

        /**
         *     在中间过程进行this赋值操作
         */
        MultiThreadsErrorThis.point = this;
        Thread.sleep(100);
        this.y = y;
    }

    @Override
    public String toString() {
        return x + ", " + y;
    }
}
```


###### 隐式对象逸出
有时候this对象不像上面的例子如此的明显, 当我们利用内部类去构造一个事件监听, 如果使用不当也会造成对象的隐式逸出。

```java

/***
 *      描述:     隐式逸出
 */
public class MultiThreadsErrorImplicit {

    int count ;

    public MultiThreadsErrorImplicit(MySource source) {

        /***
         *     1. 首先在构造函数中注册我们的事件监听, 子线程会触发此操作
         *     2. 由于我们的匿名内部类会隐式的包含外部对象(即:MultiThreadsErrorImplicit对象)的this引用,
         *        我们获取count不需要去构造对象, 直接count即可
         *     3. 注册监听事件后, 模拟构造函数还要完成其它工作等...
         */
        source.registerListener(new EventListener() {
            @Override
            public void onEvent(Event e) {
                System.out.println("\n我获取的数是: " + count);
            }
        });

        for (int i = 0; i < 10000; i++) {
            System.out.print(i);
        }

        count = 1000;
    }

    public static void main(String[] args) {
        MySource mySource = new MySource();
        new Thread(() -> {

            /***
             *  当我们的子线程去触发监听事件后, 主要是根据当前等待时间才能知道count输出的值,
             *  如果等待构造函数全部初始化完成后, 则count输出是1000, 否则count为0
             */

            try {
                Thread.sleep(10);
                // 如果等待时间较长, 构造函数后面的逻辑已经完成
//                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            mySource.eventCome(new Event() {});
        }).start();

        MultiThreadsErrorImplicit multiThreadsErrorImplicit =
                new MultiThreadsErrorImplicit(mySource);
    }


    static class MySource {
        private EventListener listener;

        void registerListener(EventListener eventListener) {
            this.listener = eventListener;
        }

        void eventCome(Event e) {
            if (listener != null)
                listener.onEvent(e);
            else
                System.out.println("还未初始化完成...");
        }
    }

    interface EventListener {
        void onEvent(Event e);
    }

    interface Event {}
}
```

###### 构造函数中新建线程

```java

/***
 *      描述:     构造函数中新建线程
 */
public class MultiThreadsErrorConstruction {

    private Map<String, String> states ;

    public MultiThreadsErrorConstruction() {

        /***
         *      当在构造函数中使用线程可能会导致取不到数据值, 当程序执行完start()方法时
         *      我们的构造函数就结束了, 而线程中的配置数据什么时候写入我们并不清楚
         *
         *      当子线程在没写入之前我们调用可能会造成空指针异常, 如果等待子线程写入后在取就不会有此问题。
         *
         *      这种根据时间等待获取是极其不稳定的, 也是不安全的。
         *
         *      但是, 我们可能在平时开发中也会在构造程序中"隐式"的用到线程, 比如数据库连接池可能底层源码会使用多线程的方式去创建。
         */

        new Thread(() -> {
            this.states = new HashMap<>();
            states.put("K1", "赛车手");
            states.put("A1", "中门对狙");
            states.put("C4", "炸弹爆炸");
            states.put("F4", "东北F4");
        }).start();
    }
}
```
