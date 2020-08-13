### Java并发工具7-同步工具

#### 概述
在学习多线程的时候, 我们可以使用join()方法来等待某个线程执行完后再执行。虽然join可以实现线程之间的同步但是join()方法也是有缺点的。

1. 需要获取到线程的引用调用join()
2. 当多个线程需要控制顺序执行join()方法会显得比较乱

当我们使用juc下面的工具包来控制线程之间的同步顺序, 多个线程使用同一个实例管理也很方便。

下面, 我们先来看看JUC给我们提供了那些线程之间同步的工具类

| 类 | 说明 |
| :-----:| :----: | 
| CountDownLatch | 多个线程相互等待, 当计数器为0时, 线程被释放 |
| CyclicBarrier | 线程之间相互等待, 当计数器为0时, 释放线程后可重新使用 |
| Semaphore | 线程在拥有信号量的"许可证"才可以执行 |
| Condition | 控制线程的等待和唤醒 |
| Phaser |  |
| Exchanger |  |


#### CountDownLatch使用

CountDownLatch类使用其实非常简单, 当我们进入CountDownLatch源码就会发现更本就没有几个方法。但是我们最主要的关注的方法有三个。

1. 构造方法只有只有, 并且需要传入等待的数量

```java
public CountDownLatch(int count) {
    //...
}
```


2. 计算器减1, 假设我们传入计数器的值为5, 那么就需要执行该方法5次, 否则等待中的线程永远都不会往下执行
```java
public void countDown() {
    // ...
}
```

3. 线程等待, 当构造方法中传入的count数量值不为0时, **哪个线程调用此方法, 哪个线程就一直是阻塞状态, 只有当count值为0才会被唤醒执行**

```java
public void await() throws InterruptedException {
    // ...
}
```


通过了解上面的API我们也知道了个大概, 当我们在主线程中调用了await()方法时, 子线程如果没有进行countDown()方法的话, 主线程就会一直阻塞, 当子线程们都执行countDown()直到0时, 就会唤醒等待中的主线程。

**这里需要注意一点: 只要没调用await()方法的线程都不会被阻塞, 多个线程调用会同时被阻塞。**


**样例1: 现在我们要去提交OA审批(申请一台显示器), 期间可能会有很多人给我们审批, 等全部审批通过之后我们才可以去运维部门领取显示器, 只要期间有一个人审批不通过我们就会一直阻塞住无法继续执行。**

```java

/***
 *
 *      描述:     CountDownLatch使用, 让主线程等待其余子线程执行完成后, 在执行。
 */
public class CountDownLatchExample01 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(5);
        ExecutorService executorService =
                Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            final int id = i + 1;
            executorService.execute(() -> {
                System.out.println("用户ID: " + id + " 正在查阅任务...");
                int r = new Random().nextInt(6) + 1;
                System.out.println("用户ID: " + id + " 正在审批任务, 大约耗时: " + r + "秒");
                try {
                    Thread.sleep(r * 1000);
                    System.out.println("用户ID: " + id + " 审批完成");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    latch.countDown();
                }
            });
        }

        System.out.println("提交任务主线程用户正在默默等待结果中...");
        latch.await();
        System.out.println("提交任务主线程用户审核通过啦!!!");

        executorService.shutdown();
    }
}
```

该例子中, 当中只要有一个线程没有执行latch.countDown()方法都会导致mian线程陷入无尽的阻塞中。


**样例2: 现在有一群学生需要等老师来了发放试卷然后开始做题, 但是在发放试卷顺序可能会造成有些同学优先做题有些学生晚一点做题, 这样就会造成时间上的不公平, 我们要做的是只有当老师说现在开始做题之后大家才能一起动笔写题。**

```java
/***
 *
 *      描述:     CountDownLatch使用, 让多个子线程等待一个主线程
 */

public class CountDownLatchExample02 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(1);
        ExecutorService executorService =
                Executors.newFixedThreadPool(5);

        for (int i = 0; i < 5; i++) {
            final int id = i + 1;
            executorService.execute(() -> {
                System.out.println("学生编号: " + id + " 领取到试卷, 等待做题...");
                try {
                    latch.await();
                    System.out.println("学生编号: " + id + " 开始做题...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        Thread.sleep(3000);     // 等待一定时间, 老师正在拿着试卷走回教室
        System.out.println("各位同学开始考试...");
        latch.countDown();
        executorService.shutdown();
    }
}
```

此例子中, 我们让所有学生领取到试卷之后才能开始写题。


至此, 对比上面两个样例, 我们发现使用CountDownLatch可以有下面两种方式:
  1. 一个线程等待多个线程执行完毕
  2. 多个线程等待一个线程执行完毕

样例1中就是一个线程等待多个线程执行完成后才能去做后续的事情, 而我们的样例2就不一样了, 它需要等待所有学生都拿到试卷后才能一起做题。

对于无论是一个线程等待多个线程还是多个线程等待一个线程取决于我们的CountDownLatch的构造参数设置, 我们要做的是更具业务逻辑调用countDown()方法将其归零即可。至于在哪里或者什么时候调用countDown()方法取决于我们自己。

对于CountDownLatch类, 我们当然还可以混用, 如下例子:

**样例3: 每个学生写完并提交试卷的时间都是不一样的, 有些人很快有些人很慢, 老师需要等到最后一名同学提交试卷后才会走, 这个时候就需要老师等待学生了。**

```java
/***
 *
 *      描述:     CountDownLatch使用, 多个子线程等待主线程以及主线程等待多个子线程
 */
public class CountDownLatchExample03 {

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch teach = new CountDownLatch(1);       // 老师的门闩
        CountDownLatch student = new CountDownLatch(5);     // 学生的门闩

        ExecutorService executorService =
                Executors.newFixedThreadPool(5);

        for (int i = 0; i < 5; i++) {
            final int id = i + 1;
            executorService.execute(() -> {
                System.out.println("学生编号: " + id + " 领取到试卷, 等待做题...");
                try {
                    teach.await();
                    System.out.println("学生编号: " + id + " 开始做题...");
                    int r = new Random().nextInt(10) + 1;
                    Thread.sleep(r * 1000);
                    System.out.println("学生编号: " + id + " 提交试卷");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    student.countDown();
                }
            });
        }

        Thread.sleep(3000);
        teach.countDown();  //  开始考试

        student.await();    // 等待学生都提交试卷
        System.out.println("所有学生都提交了试卷....");
        executorService.shutdown();
    }
}
```

此例中可以发现, 学生既要等老师说开始考试才能开始做题, 老师也需要等到最后一个学生提交试卷后才能离开。使用两个CountDownLatch进行配和。

对于CountDownLatch也是有缺点的:
  1. CountDownLatch是不可以复用的, 当我们调用过await()方法后, 如果还是同一实例的CountDownLatch则无效不会等待。