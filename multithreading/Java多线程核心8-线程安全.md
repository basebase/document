#### Java多线程核心8-线程安全

##### 什么是线程安全?
在写这个之前, 我会引用一些我觉得还不错的文章或者书籍等, 然后我们来看看每条引用有什么不同之处。

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
