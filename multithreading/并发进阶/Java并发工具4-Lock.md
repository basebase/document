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
