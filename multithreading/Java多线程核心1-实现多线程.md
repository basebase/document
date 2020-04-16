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
