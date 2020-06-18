#### Java并发工具3-ThreadLocal

##### ThreadLocal能干嘛?

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
