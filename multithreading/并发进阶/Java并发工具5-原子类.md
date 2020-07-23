### Java并发工具4-原子类

#### 概述
Atomic(原子)操作一般认为是最小的单位。一段代码如果是原子的, 则表示这段代码在执行过程中要么成功, 要么失败。
原子操作一般都是底层通过CPU的指令来实现。而java.util.concurrent.atomic包下的类, 可以让我们在多线程环境下,
通过一种无锁的原子方式实现线程安全。

比如, 我们常说的i++或者++i在多线程环境下就不是线程安全的, 因为这一段代码包含了三个独立的操作, 在没有使用原子变量之前
我们可以通过加锁才能保证"读->改->写"这三个操作时的"原子性"。


#### Java Atomic原子类纵览

|  类型   | 类  |
|  ----  | ----  |
| Atomic基本类型  | AtomicInteger, AtomicLong, AtomicBoolean|
| Atomic数组类型  | AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray |
| Atomic引用类型  | AtomicReference, AtomicStampedReference, AtomicMarkableReference |
| Atomic升级类型  | AtomicIntegerFieldUpdater, AtomicLongFieldUpdater, AtomicReferenceFieldUpdater |

在JDK8之前只有上面这四种类型, 但是JDK8之后新增了两种累加器。

|  类型   | 类  |
|  ----  | ----  |
| Adder累加器  | LongAdder, DoubleAdder|
| Accumulator累加器  | LongAccumulator, DoubleAccumulator|
