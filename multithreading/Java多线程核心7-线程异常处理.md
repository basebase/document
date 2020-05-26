#### Java多线程核心7-线程异常处理

##### Java线程异常处理

平常我们只有单线程(只有一个main线程)的程序, 在遇到一个没有捕获的异常后会抛出异常信息并终止程序应用程序。但是如果我们有多个线程的情况下呢? 又会出现什么样的结果?


我们带着下面的问题来学习:
  * 在多线程中子线程抛出异常, 会终止程序吗?
  * 在多线程中子线程抛出异常, 在调用者线程进行try/catch会发生什么?
  * 如何处理子线程异常?




我们先来思考一个问题, 当我们在main线程中启动一个会发生异常的子线程, 那么此时我们的main线程会发生什么? 程序会终止吗? 还是会继续执行直到main线程逻辑执行完成退出?

```java
/***
 *      描述:     如果只有main线程的程序, 会抛出异常异常信息。
 *               但是, 如果是多线程, 子线程发生异常, 会出现什么情况?
 */
public class ExceptionInChildThread {

    public static void main(String[] args) {

        /***
         *     可以看到, 子线程如果发生异常信息, 会把异常信息抛出来, 但是不影响主线程main的执行
         *     我们可以看到控制台下, main线程依旧把循环体内容全部输出, 而子线程的异常信息会沉没在一大串内容中...
         */

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " SS ");
            int i = 1 / 0;
            System.out.println(Thread.currentThread().getName() + " GG ");
        }).start();

        for (int i = 0; i < 1000; i++) {
            System.out.println(i);
        }
    }
}
```

**可以看到, 子线程会抛出异常信息, 但是不会终止程序, main线程继续执行直到执行完成后退出。**


上面的程序, 我们没有去捕获异常信息, 那我们在main线程中捕获异常信息呢?是不是会终止程序呢?

```java
/***
 *      描述:     我们启动四个会抛出异常的线程程序, 在main线程(调用者)中进行捕获, 会发生什么情况?
 *                  1. 捕获到第一个线程异常后, 其余线程都不应该运行了, 并且打印出异常信息
 *                  2. main线程根本没有捕获到异常, 其余线程依旧运行并抛出异常信息
 */
public class CantCatchDirectly {

    public static void main(String[] args) {

        try {

            /***
             *      从运行结果上来看, main线程中并没有执行到catch中的代码, 代表子线程的异常在main线程没捕获到
             *      其余线程依旧执行并抛出各自的异常信息, 这是为什么呢?
             *
             *      每个线程可以想象成一个人, main线程是总负责人, 其余的子线程是工作人员, 而main线程说大家都去工作吧调用start()方法
             *      每个子线程都各自去完成自己的工作, 期间完成的怎么样main线程并不清楚, 而只有在main线程中不小心让一个工作线程启动1次以上
             *      start()方法, 这个时候main线程就会捕获到异常并终止程序。
             *
             *      总结下:
             *              每个线程都是独立的, 所以main线程中使用try/catch捕获子线程异常是无法捕捉到的,
             *              try/catch只能捕获对应线程内的异常(而当前对应的线程是main线程中发生的异常信息)
             *
             */

            new Thread(() -> {
                int i = 1 / 0;
            }).start();

            new Thread(() -> {
                int i = 1 / 0;
            }).start();

            new Thread(() -> {
                int i = 1 / 0;
            }).start();

            new Thread(() -> {
                int i = 1 / 0;
            }).start();
        } catch (ArithmeticException e) {
            System.out.println("检测到子线程异常啦!!!" + e);
            e.printStackTrace();
        }
    }
}
```

很显然, 上面的程序即使我们在main线程中捕获了异常信息, 但是就和没有捕获一样, 各个线程依旧执行着自己的逻辑并抛出异常信息, 所以在调用者线程中进行捕获其它线程是无效的...


上面两个程序我们都无法直接捕获子线程异常, 可能就会导致子线程会抛出异常信息, 而我们却没有感知到异常的发生, 那么如何处理这种问题呢?

我们可以通过UncaughtExceptionHandler该线程全局异常处理类来进行捕获子线程的异常信息。
