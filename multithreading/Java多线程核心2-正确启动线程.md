#### Java多线程核心2-正确启动线程

##### 如何正确启动线程
我们在创建的时候无论是继承Thread还是实现Runnable接口, 都会实现一个run()方法,
当我们创建线程并启动线程, 最终都会执行run()方法, 思考下面几个问题:

* 调用start()方法和调用run()方法都可以启动线程吗?
* 执行一次以上start()方法会有什么问题?
* start()方法和run()方法的区别?


###### start和run

通过下面一个例子来观察一下当调用start和run方法会有什么结果。

```java
/***
 *      描述:     对比start和run方法两种启动线程的方式
 */
public class StartAndRunMethod {
    public static void main(String[] args) {
        Runnable runnable = () -> {
          System.out.println(Thread.currentThread().getName());
        };

       runnable.run();
       new Thread(runnable).start();
    }
}
```

输出结果
```text
main
Thread-0
```

可以看到结果输出一个为main线程, 一个为Thread-0线程。
所以, 直接调用run()方法是没有创建一个新的线程的, 而必须要使用start()方法才能启动一个线程。


##### 调用两次start

现在, 我们线程创建好了, 也知道如何正确的启动一个线程。那么, 我们可以多次启动同一个线程吗?

```java
/***
 *      描述:     调用两次start()方法会有什么问题
 */
public class CanStartTwice {
    public static void main(String[] args) {
        Thread t = new Thread();
        t.start();
        t.start();
    }
}
```
当我们执行这一段代码的时候, 是执行正常呢?还是执行异常呢?
下面就是其输出结果:
```text
Exception in thread "main" java.lang.IllegalThreadStateException
	at java.lang.Thread.start(Thread.java:708)
	at com.moyu.example.multithreading.ch02.CanStartTwice.main(CanStartTwice.java:10)
```

可以看到抛出了异常信息。为什么呢?
