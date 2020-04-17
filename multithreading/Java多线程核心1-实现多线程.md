#### Java多线程核心1-实现多线程


##### 如何创建新线程?

###### 实现多线程究竟有几种方法?

我们先列举一下有哪些方式能创建多线程
  1. 继承Thread类
  2. 实现Runnable接口, 重写run方法
  3. 实现Callable接口, 重写call方法
  4. 使用线程池
  5. 定时器
  6. 匿名内部类

可以看到, 有各种各样实现线程的方式, 但是我们看一下oracle官方文档究竟有几种实现多线程的方式。

[Oracle官方文档](https://docs.oracle.com/javase/8/docs/api/)

Oracle在介绍线程的时候, 说明了只有两种方式创建线程

**"There are two ways to create a new thread of execution"**

**One is to declare a class to be a subclass of Thread. (将类声明为Thread子类)**

**The other way to create a thread is to declare a class that implements the Runnable interface (声明一个实现Runnable接口的类)**

所以, 我们实现多线程只有两种方式
  * 一种是继承Thread类
  * 另外一种是实现Runnable接口


###### 实现多线程创建

[实现多线程代码](https://github.com/basebase/java-examples/tree/master/src/main/java/com/moyu/example/multithreading/ch01)

```java
/***
 *  描述:   实现Runnable接口创建线程
 */
public class RunnableStyle implements Runnable {

    @Override
    public void run() {
        System.out.println("Thread Name : " + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        RunnableStyle runnableStyle = new RunnableStyle();
        for (int i = 0 ; i < 10; i ++) {
            Thread t = new Thread(runnableStyle);
            t.start();
        }
    }
}
```

```java
/***
 * 描述:     利用Thread实现创建线程
 */
public class ThreadStyle extends Thread {

    @Override
    public void run() {
        System.out.println("利用Thread实现多线程: " + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i ++)
            new ThreadStyle().start();
    }
}
```



##### 两种方法对比

参考:

[Difference between Thread vs Runnable interface in Java](https://javarevisited.blogspot.com/2012/01/difference-thread-vs-runnable-interface.html#ixzz6JlXYyMxZ)  
[JAVA多线程之Runnable和Thread比较](https://zhuanlan.zhihu.com/p/32362557)  
[“implements Runnable” vs “extends Thread” in Java](https://stackoverflow.com/questions/541487/implements-runnable-vs-extends-thread-in-java#)

实现一个Runnable接口和继承Thread类, 我们用哪种方式会更好呢?

也有很多人讨论此问题, 更多的是更倾向于使用Runnable接口的方式。

那我们也整理一下, 为什么使用Runnable接口而非继承Thread类。
1. 实现Runnable接口可以避免继承Thread类, 如果继承了Thread类此后便无法扩展任何其它类。(Java不支持多继承)

2. 从设计上来看, 使用Runnable相当于一个任务。我们可以重用该任务。达到解耦作用。

其中在stackoverflow上有一句话

**However, one significant difference between implementing Runnable and extending Thread is that
by extending Thread, each of your threads has a unique object associated with it, whereas implementing Runnable, many threads can share the same object instance.**

大致的意思是: 实现Runnable和扩展Thread之间的一个重要区别是通过扩展Thread，您的每个线程都有一个与之关联的唯一对象，而实现Runnable时，许多线程可以共享同一对象实例。


并提供了以下的一个例子:

```java
//Implement Runnable Interface...
class ImplementsRunnable implements Runnable {

private int counter = 0;

public void run() {
    counter++;
    System.out.println("ImplementsRunnable : Counter : " + counter);
 }
}

//Extend Thread class...
class ExtendsThread extends Thread {

private int counter = 0;

public void run() {
    counter++;
    System.out.println("ExtendsThread : Counter : " + counter);
 }
}

//Use the above classes here in main to understand the differences more clearly...
public class ThreadVsRunnable {

public static void main(String args[]) throws Exception {
    // Multiple threads share the same object.
    ImplementsRunnable rc = new ImplementsRunnable();
    Thread t1 = new Thread(rc);
    t1.start();
    Thread.sleep(1000); // Waiting for 1 second before starting next thread
    Thread t2 = new Thread(rc);
    t2.start();
    Thread.sleep(1000); // Waiting for 1 second before starting next thread
    Thread t3 = new Thread(rc);
    t3.start();

    // Creating new instance for every thread access.
    ExtendsThread tc1 = new ExtendsThread();
    tc1.start();
    Thread.sleep(1000); // Waiting for 1 second before starting next thread
    ExtendsThread tc2 = new ExtendsThread();
    tc2.start();
    Thread.sleep(1000); // Waiting for 1 second before starting next thread
    ExtendsThread tc3 = new ExtendsThread();
    tc3.start();
 }
}
```

输出结果:

```text
ImplementsRunnable : Counter : 1
ImplementsRunnable : Counter : 2
ImplementsRunnable : Counter : 3
ExtendsThread : Counter : 1
ExtendsThread : Counter : 1
ExtendsThread : Counter : 1
```

其实就结果来说, 代码本身没有问题, 但是测试的方法却有问题。大家可以看参考中的
**“implements Runnable” vs “extends Thread” in Java**  
@zEro的评论, 很有意思。

如果, 我们在创建ExtendsThread的时候只创建一个对象, 然后再用Thread启动呢? 可以发现数据一样是被共享的。

```java
ExtendsThreadV2 et = new ExtendsThreadV2();

Thread tc1 = new Thread(et);
tc1.start();
Thread.sleep(1000);

Thread tc2 = new Thread(et);
tc2.start();
Thread.sleep(1000);

Thread tc3 = new Thread(et);
tc3.start();
```

输出结果:

```text
ExtendsThread : Counter : 1
ExtendsThread : Counter : 2
ExtendsThread : Counter : 3
```
所以资源共享, 我觉得说只有Runnable能实现显然不是很能让人信服。当然可看到很多人写文章说明为什么Runnable可以资源共享而Thread不行, 但文章基本大同小异。[欢迎大家在评论区留言讨论。]


###### 同时使用Thread和Runnable
实现线程要么使用Thread, 要么使用Runnable来实现。两种一起上[实际开发不会出现]?会是什么一个效果?

请看下面这个例子:

```java

/***
 *   描述:      同时使用Runnable和Thread两种方式实现线程
 */
public class BothRunnableThread {
    public static void main(String[] args) throws Exception {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Runnable run()...");
            }
        }) {
            @Override
            public void run() {
                System.out.println("Thread run() ...");
            }
        }.start();
    }
}
```

在运行代码之前能正确的说出输出结果吗?
在说出结果之前, 我们先来看看Thread类的run方法是如何实现的。

```java
// Thread.java
private Runnable target;

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

可以看到, 我们在构造Thread类的时候, 如果传入Runnable对象就用Runnable对象的run方法, 就是这么简单的一句话。但是, 如果我们继承Thread类并重写其run()方法, 那么就不会使用上面的run()方法, 而是我们自定义的方法。

通过上面的了解, 可得:
* Runnable方法: 最终调用target.run()
* Thread方法: run()方法被重写

那么, 在了解了Thread和Runnable调用方法的区别后, 可以正确的输出答案吗?

```text
Thread run() ...
```
对, 因为Thread类重写了run()方法, 所以Runnable是不会被调用的。


##### 多种线程创建方式分析
经过上面的学习, 已经知道如何创建一个线程了。那么, 为什么会有这么多创建线程的方式呢?

其实可以发现, 无论是线程池的方式创建还是匿名内部类或者实现Callable接口又或者是
定时器。他们最终都是创建Thread对象或者实现Runnable接口的方式。在比如说JDK8的lambda表达式也可以创建一个线程, 那这也算一种创建方式吗?

所以, 可能创建线程有很多"姿势", 但是归其本质就是Thread和Runnable。

##### 总结
1. 创建线程的方式只有两种
2. Thread和Runnable方式对比, 以及应当应当使用哪种方式更为优雅。
3. 为什么会出现多种创建线程的方式, 分析多种创建线程方式本质。
4. [本章代码](https://github.com/basebase/java-examples/tree/master/src/main/java/com/moyu/example/multithreading/ch01)
