#### Java并发工具4-Lock

##### 概述
对于Java中的锁, 不仅仅是我们之前学习的synchronized关键字。其实juc还提供了Lock接口, 并实现了各种各样的锁。每种锁的特性不同, 可以适用于不同的场景展示出非常高的效率。

由于锁的特性非常多, 故按照特性进行分类, 帮助大家快速的梳理相关知识点。

![java锁脑图](https://github.com/basebase/img_server/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B/java%E9%94%81.png?raw=true)

我们会从下面路径进行学习
  * 乐观锁&悲观锁
  * 可重入锁&非可重入锁
  * 公平锁&非公平锁
  * 共享锁&排它锁
  * 自旋锁&阻塞锁
  * 锁优化


##### Lock还是synchronized?

###### Lock

我们先来看看JDK对Lock的描述:

**Lock implementations provide more extensive locking operations than can be obtained using synchronized methods and statements**

Lock提供比synchronized更广泛的锁操作。

**A lock is a tool for controlling access to a shared resource by multiple threads. Commonly, a lock provides exclusive access to a shared resource**

Lock是一种用于控制多个线程对共享资源独占访问的工具。

**only one thread at a time can acquire the lock and all access to the shared resource requires that the lock be acquired first. However, some locks may allow concurrent access to a shared resource, such as the read lock of a ReadWriteLock.**

一次只有一个线程可以获取到Lock锁, 并且对共享资源的所有访问都需要先获取到锁。
但是, 某些锁可能允许并发访问共享资源, 例如: ReadWriteLock读写锁。

当然, 更多的描述可以自行查阅JDK的文档内容。不过从这里的信息也大概了解到, Lock要做的事情其实是和synchronized一样的, 都是让多个线程获取到锁才能操作共享资源。
只不过, Lock能带来一些灵活的操作, 但是灵活的代价就是维护的一些成本, 后面会说到。


###### synchronized
对Lock有了一个大概的了解后, 那synchronized呢?

鉴于很多地方在学习Lock都会对比synchronized。那就必须要看看使用synchronized会有哪些问题? Lock工具类是否又帮助我们解决了?


使用synchronized保护一段被多个线程访问的代码块, 当一个线程获取到锁后, 其它线程只能等待锁被释放, 而释放锁只有下面两种情况:
  1. 获取锁的线程执行完了同步代码块内容, 正常的释放持有的锁;
  2. 线程执行发生异常, 此时JVM让线程自动释放锁;

假设, 当前获取到锁的线程等待I/O或者长时间sleep被阻塞, 但是锁没有被释放, 其它线程只能安静的等待, 这是非常影响程序的执行效率的。

因此, 就需要一种机制让那些等待的线程别无期限的等待下去了(比如只等待一定时间或者响应中断), 通过Lock就可以实现。


在比如说, 当有多个线程读写文件时, 读操作和写操作会发生冲突, 写操作和写操作也会发生冲突, 但是读操作和读操作不会发生冲突。

如果使用synchronized关键字来实现同步, 就会导致一个问题, 多个线程都是读操作, 当一个线程获取到锁时进行只读操作, 其余线程只能等待而无法进行读操作。

因此, 需要一种机制来使得多个线程都是进行读操作时, 线程之间不会发生冲突, 通过Lock可以实现。

最后, 通过Lock接口可以清楚的知道线程有没有成功获取到锁, 而synchronized则无法办到。


总结一下:
  1. Lock不是Java内置的, synchronized是Java关键字, 因此是内置特性。Lock是一个类, 通过这个类可以实现同步访问;

  2. Lock和synchronized有一点非常不同, 使用synchronized不需要手动释放锁, 当synchronized方法或者synchronized代码块执行完后, 系统会自动让线程释放对锁的占用; 而Lock则必须要手动释放锁, 如果没有释放锁, 可能会出现死锁。

  3. 至于使用Lock还是synchronized, 如果只锁定一个对象还是建议使用synchronized, 使用Lock则需要自己手动释放锁等。
