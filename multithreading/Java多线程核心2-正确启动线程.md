#### Java多线程核心2-正确启动线程

##### 如何正确启动线程
我们在创建的时候无论是继承Thread还是实现Runnable接口, 都会实现一个run()方法,
当我们创建线程并启动线程, 最终都会执行run()方法, 思考下面几个问题:

* 调用start()方法和调用run()方法都可以启动线程吗?
* 执行一次以上start()方法会有什么问题?
* start()方法和run()方法的区别?


###### start和run

通过下面一个例子来观察一下当调用start和run方法会有什么结果。

```java
/***
 *      描述:     对比start和run方法两种启动线程的方式
 */
public class StartAndRunMethod {
    public static void main(String[] args) {
        Runnable runnable = () -> {
          System.out.println(Thread.currentThread().getName());
        };

       runnable.run();
       new Thread(runnable).start();
    }
}
```

输出结果
```text
main
Thread-0
```

可以看到结果输出一个为main线程, 一个为Thread-0线程。
所以, 直接调用run()方法是没有创建一个新的线程的, 而是当成一个普通方法执行了, 而使用start()方法才能启动一个新线程。


##### 调用两次start

现在, 我们线程创建好了, 也知道如何正确的启动一个线程。那么, 我们可以多次启动同一个线程吗?

```java
/***
 *      描述:     调用两次start()方法会有什么问题
 */
public class CanStartTwice {
    public static void main(String[] args) {
        Thread t = new Thread();
        t.start();
        t.start();
    }
}
```
当我们执行这一段代码的时候, 是执行正常呢?还是执行异常呢?
下面就是其输出结果:
```text
Exception in thread "main" java.lang.IllegalThreadStateException
	at java.lang.Thread.start(Thread.java:708)
	at com.moyu.example.multithreading.ch02.CanStartTwice.main(CanStartTwice.java:10)
```

如果执行两次run()方法肯定不会抛出任何异常信息, 毕竟直接调用run()就和调用普通方法是一样的。那么为什么会抛出异常呢?

##### start vs run 内部

上面提到, 为什么调用两次start()方法会抛出异常呢?我们先来看下start()方法内部如何实现的。

```java
// Thread.java

private volatile int threadStatus = 0;
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
    }
}

private native void start0();
```
可以看到start()方法正式通过threadStatus变量来判断多次启动, 初始值为0表示NEW状态。(线程状态后面会介绍)并将该线程加入线程组中, 随即调用start0()方法, 该方法是一个本地方法, 有兴趣的话可以看看Open JDK源码。

我这里用的是JDK8, 对应Open JDK C/C++文件链接为:

[Thread.c](https://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/f0b93fbd8cf8/src/share/native/java/lang/Thread.c)

```c
static JNINativeMethod methods[] = {
    // start0就是我们要找的方法。

    {"start0",           "()V",        (void *)&JVM_StartThread},
    {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
    {"isAlive",          "()Z",        (void *)&JVM_IsThreadAlive},
    {"suspend0",         "()V",        (void *)&JVM_SuspendThread},
    {"resume0",          "()V",        (void *)&JVM_ResumeThread},
    {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
    {"yield",            "()V",        (void *)&JVM_Yield},
    {"sleep",            "(J)V",       (void *)&JVM_Sleep},
    {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
    {"countStackFrames", "()I",        (void *)&JVM_CountStackFrames},
    {"interrupt0",       "()V",        (void *)&JVM_Interrupt},
    {"isInterrupted",    "(Z)Z",       (void *)&JVM_IsInterrupted},
    {"holdsLock",        "(" OBJ ")Z", (void *)&JVM_HoldsLock},
    {"getThreads",        "()[" THD,   (void *)&JVM_GetAllThreads},
    {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
    {"setNativeName",    "(" STR ")V", (void *)&JVM_SetNativeThreadName},
};
```


[jvm.cpp](https://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/76a9c9cf14f1/src/share/vm/prims/jvm.cpp)

```c++

JVM_ENTRY(void, JVM_StartThread(JNIEnv* env, jobject jthread))
  JVMWrapper("JVM_StartThread");
  JavaThread *native_thread = NULL;
{
  MutexLocker mu(Threads_lock);

  if (java_lang_Thread::thread(JNIHandles::resolve_non_null(jthread)) != NULL) {
    throw_illegal_thread_state = true;
  } else {
    itself when it starts running

    jlong size =
           java_lang_Thread::stackSize(JNIHandles::resolve_non_null(jthread));
    size_t sz = size > 0 ? (size_t) size : 0;

    // 主要看此方法
    native_thread = new JavaThread(&thread_entry, sz);

    if (native_thread->osthread() != NULL) {
      native_thread->prepare(jthread);
    }
  }
}


static void thread_entry(JavaThread* thread, TRAPS) {
  HandleMark hm(THREAD);
  Handle obj(THREAD, thread->threadObj());
  JavaValue result(T_VOID);
  JavaCalls::call_virtual(&result,
                          obj,
                          KlassHandle(THREAD, SystemDictionary::Thread_klass()),

                          // 可以看到这里传入了run方法, 所以start()方法会调用run()方法
                          vmSymbols::run_method_name(),
                          vmSymbols::void_method_signature(),
                          THREAD);
}

```

**ps: 本人不会C/C++, 也是参考别人的博文。**


当我们在一次的调用start()方法, threadStatus变量的值不在为0直接抛出异常。







##### 结尾

##### 总结
1. run方法不能启动线程, 而被当成一个普通方法执行。而start方法可以启动线程并执行run方法。
2. 一个线程不能多次调用start()方法, 否则抛出异常。
3. 了解多次调用start方法为什么抛出异常, 简单了解线程状态。
4. 了解Open JDK源码, 了解start0()方法是如何调用run方法的。

##### 本节源码
[Java多线程核心2-正确启动线程源码](https://github.com/basebase/java-examples/tree/master/src/main/java/com/moyu/example/multithreading/ch02)

###### 参考:
1. [Difference between start and run method in Thread – Java Tutorial and Interview Question](https://javarevisited.blogspot.com/2012/03/difference-between-start-and-run-method.html)
2. [Difference between Thread.start() and Thread.run() in Java
](https://www.geeksforgeeks.org/difference-between-thread-start-and-thread-run-in-java/)
3. [java中多线程执行时，为何调用的是start()方法而不是run()方法](https://www.cnblogs.com/heqiyoujing/p/11355264.html)
4. [Java多线程中start()和run()的区别](https://www.cnblogs.com/sunflower627/p/4816821.html)
5. [Java线程Run和Start的区别](https://www.cnblogs.com/alinainai/p/10389161.html)
