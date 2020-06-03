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

修复2:
