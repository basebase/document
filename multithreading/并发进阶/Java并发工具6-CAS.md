### Java并发工具6-CAS

#### 概述
CAS的全拼为(compare and swap)中文一般称为比较并交换, 它是原子操作的一种。可用于在多线程中实现不被打断的数据交互操作, 从而避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题。

该操作通过将内存的值与指定数据进行比较, 当数值一致时将内存中的数据值替换为新的值。

通常来说, 当有多个线程去修改一个值的时候, 只有一个线程可以修改程序, 其余的线程都会失败。


#### CAS实现原理
之前学习过的锁章节中, 有悲观锁和乐观锁, 而我们的乐观锁就是基于CAS实现的。但是当时并没有具体介绍什么是CAS。
CAS又是如何更新我们的值呢?

CAS执行有三步:
  * 从内存地址X读取值V(之后要更新的值)
  * 预期值A(上一次从内存中读取的值)
  * 新值B(写入到V上的值)

当从理论上来看或许会有点晕乎, 什么V什么A和B, 都是写啥?

这里先用伪代码来分解CAS的流程, 之后会通过Java来模拟一个CAS执行流程。

假设我们有个整形变量V为10, 当前有N个线程想要递增该值并用于其它操作。让我们来看看CAS是如何执行的:

1) 现在有线程1和线程2都想要递增V, 它们读取值之后将其增加到11
```java
V = 10, A = 0, B = 0
```

2) 假设线程1首先执行, V将于线程1读取到的值进行比较
```java
V = 10, A = 10, B = 11

if V == A
  V = B
else
  operation failed
  return V
```

很显然, V的值将被覆盖为11, 操作成功。

3) 此时线程2到来并尝试与线程1相同的操作

```java
V = 10, A = 10, B = 11
if V == A
  V = B
else
  operation failed
  return V
```
这种情况下, A的值不等V, 因此不替换值。并返回V当前的值, 即11。线程2此时再次使用获取到的值进行重试

4) 线程2再次执行
```java
V = 11，A = 11，B = 12
```
这一次的A的值是和V相等的。增量值为12, 返回线程2。



当我们使用 
```java
if V == A
```
这个操作就是CAS的第一步和第二步。从内存地址读取值V和我们的预期值A进行比较, 如果一致就会执行第三步更新变量值V
```java
V = B
```

上面就是一个CAS的执行流程, 简而言之, 当多个线程尝试使用CAS同时更新一个变量时, 其中一个线程更新成功其余线程都将失败。
其余更新失败的线程可以进行重试又或者什么都不处理。


#### 参考
[Java Compare and Swap Example – CAS Algorithm](https://howtodoinjava.com/java/multi-threading/compare-and-swap-cas-algorithm/)

[Java 101: Java concurrency without the pain, Part 2](https://www.infoworld.com/article/2078848/java-concurrency-java-101-the-next-generation-java-concurrency-without-the-pain-part-2.html?page=3)

[非阻塞同步算法与CAS(Compare and Swap)无锁算法](https://www.cnblogs.com/mainz/p/3546347.html)

[并发编程—CAS（Compare And Swap）](https://segmentfault.com/a/1190000015239603)