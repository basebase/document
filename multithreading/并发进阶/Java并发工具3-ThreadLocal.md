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
