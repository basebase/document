### flink入门-水位线图解

#### 水位线图解
上一小节介绍了watermark的概念, 可能还是会比较晦涩, 这一小节通过图的方式进行介绍。

![flink水位线图解-1](https://github.com/basebase/document/blob/master/flink/image/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF%E5%9B%BE%E8%A7%A3-1.png?raw=true)

如上图, 图中有一个队列, 里面存放了一些数据, 圆圈可以理解为一个事件, 里面的数字可以理解为事件发生的时间戳, 而三角形则是watermark。
可以看到的是队列中的数据是一个乱序的存在, 现在我们的需求是5s为一个窗口计算, 最大延迟时间可以为2s, 看看是如何划分的。

这里Flink是如何计算watermark的值呢?

***watermark = 最大事件时间戳 - 延迟时间***

那么有watermark的窗口(window)在什么时候触发窗口计算呢?

***窗口停止时间 <= 当前watermark***

下面这张图展示了将数据划分到对应的窗口以及如何生成watermark和触发计算窗口

![flink水位线图解-2](https://github.com/basebase/document/blob/master/flink/image/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF/flink%E6%B0%B4%E4%BD%8D%E7%BA%BF%E5%9B%BE%E8%A7%A3-2.png?raw=true)

当时间戳为1的数据进入将其划分到[0,5)的窗口, 但是当前watermark并没有太多意义, 当时间戳为4的数据进入后将其划分到[0,5)的窗口并将watermark更新为2, 当时间戳为5的数据进入后, 由于是前闭后开的区间所有不能将数据放置[0, 5)的窗口而是放入一个新窗口为[5, 10)中并将watermark更新为3, 以此类推..., 当时间戳为7的数据进入后, 当前watermark更新为5, 表示时间戳为5之前的数据都到齐了, 而我们正好有一个[0,5)的窗口就被触发计算了。

假设之后还有窗口[0, 5)的数据进入, 如果没有其它处理则该条数据会被丢弃。

通过图解, 我们在来回顾一下watermark两个基本特性

watermark时间戳一定是递增不会缩小的, 如果当前事件时间戳减去延迟时间不大于当前watermark则watermark不进行更新, 并且我们认为后续事件不会再出现有比当前watermark更小的时间戳数据。



#### 参考
[Flink 轻松理解Watermark](https://cloud.tencent.com/developer/article/1481809)

[Flink 基础学习(九) 再谈 Watermark](http://www.justdojava.com/2019/12/23/flink_learn_watermark/)