#### Java多线程核心4-线程生命周期

##### Java多线程状态(理论)
当我们在创建一个线程, 启动一个线程并执行一个线程, 那么, 这个线程究竟经过了哪些状态的转变？
线程又有多少种状态呢?

那么, 我们首先来看一下, 线程究竟有多少种状态。这里, 我直接将Thread类的内部类信息复制出来。

```java
// Thread.Java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

Thread类定义一个枚举类来描述线程状态信息, 线程一共有6种状态。

那么线程这6种状态如何去理解呢?

1. 初始状态(NEW)
  - 新创建一个线程对象, 但还没有调用start()方法。此时的线程就已经进入了初始状态

2. 运行状态(RUNNABLE)
这里会比较复杂一点, Java线程将就绪(ready)和运行中(running)两种状态笼统的称为"运行"。
  - 就绪状态(就绪状态表示可以执行, 但是还没轮到你, 你就先在这等着 )
  - 运行中状态

3. 阻塞状态(BLOCKED)
  - 在进入synchronized关键字修饰的方法或者代码块时的状态

4. 等待状态(WAITING)
  - 该状态下线程不会被执行, 它们需要被显示的唤醒, 否则会处于无限期的等待状态

5. 超时等待(TIMED_WAITING)
  - 该状态和WAITING状态很像, 只不过它有一个时间期限, 如果没有被显示的唤醒, 只需要等到设置时间超过后会被自动唤醒。

6. 终止(TERMINATED)
  - 当线程的run()方法完成时, 当然也包括异常终止。


附上一张Java线程声明周期图
![Java线程生命周期](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.png?raw=true)
