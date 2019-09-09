### 数组

#### 认识数组
数组, 是我们学习数据结构最基础的一种了, 无论是什么语言都有数组。
我们又如何去理解数组呢? 数组有哪些特点呢?

##### 理解数组
  * 数组是由相同类型的元素的集合所组成的数据结构
  * 连续的内存来存储
  * 利用元素的索引(index)获取对应值, 索引值从0开始
  * 请求内存空间后大小固定, 不能在改变
  * 数组可以存在一维二维甚至更多

通过上面3个维度, 我们可以了解一个数组原来是通过上面三个维度构建的。或许在初学的时候还是觉得比较抽象
这些都是什么鬼啊...

做个简单比喻, 现在我们去一家超市, 超市里面有柜子让我们存放自己的物品, 现在我们就把这个柜子看成是一个数组
一个大柜子包含很多小格子, 每个小格子是不是相连在一起的[这里就指的连续内存存放], 我们柜子不可能就只有一排吧。
还有2排, 3排等...这里就是一个二维数组, 那我们把东西放进去之后, 是不是有个钥匙编号(index), 通过编号我们
是不是能快速找到我们自己存放东西的盒子。那么最后, 超市说: 我这里的柜子只能放手机其余的都不让放[相同类型]
那么我们的衣服, 包等物品都无法放入进去...


那么, 程序中又是怎么定义一个数组呢?

程序中也分几步
  * 数组类型 + 中括号
  * 数组名称
  * 如果能直接定义的话, 就直接写{"需要的内容"}, 如果是初始化需要new [长度]

```java
String[] names = {"xiaomoyu", "666", "999"}; // 直接写入, 我们知道就是这些值
int[] scores = new int[3]; // 还不知道后面要写什么, 先定义好
scores[0] = 1;
scores[1] = 2;
scores[2] = 3;
```


#### 自定义数组
经过上面的学习, 我们已经了解到数组的基本使用, 但是我觉得就是这么使用不是我想要的, 我想要对数组做一些封装。


那么我从下面几个方面介绍一下:
  * 数组的容量
  * 数组的实际存储元素量
  * 默认数组容量
  * 数组是否为空
  * 获取数组容量
  * 获取数组实际存储元素数量
  * 增删改查
  * 动态扩展



**在新增的时候, 我们需要注意, size是我们维护的一个元数据信息, 当我们向数组添加元素的时候, size就进行++操作,
当size超出数组容量的时候就要抛出异常。

如果, 我们想指定位置上添加元素呢?

例如:

  我们数组长度为5, 我们已经写入了 66 77 88在数组的0,1,2的位置, 现在我想在数组1的位置加入值为100的数据
  我要怎么处理呢?


  我们只有把已存在的元素往后面移动, 现在的size = 3 我们需要把88放入下标4, 77放入下标3, 66放入下表2



**




我们来创建一个Array类来实现上述功能
```java
class Array {
  private int[] data ;
  private int size ;

  private static final int DEFAULT_CAPACITY = 10;

  /**
    用户自定义一个数组容量, 默认实际元素为0
  */
  public Array(int capacity) {
    this.data = new int[capacity];
    this.size = 0;
  }

  /**
    创建默认数组, 默认数组大小为10
  */
  public Array() {
    this(DEFAULT_CAPACITY);
  }

  /**
    判断数组是否为空
  */
  public boolean isEmpty() {
    return this.size == 0 ;
  }

  /**
    获取数组的容量大小
  */
  public int getCapacity() {
    return this.data.length;
  }

  /**
    获取实际元素
  */
  public int getSize() {
    return this.size ;
  }


  /**
    想数组头部添加一个元素
  */
  public void addLast(int e) {
    if (size == this.data.length) {
      throw new IllegalArgumentException("Add failed. Array is full.")
    }

    // 添加元素
    this.data[size] = e;
    size ++;
  }

  public void add(int index, int e) {
    if (size == this.data.length) {
      throw new IllegalArgumentException("Add failed. Array is full.")
    }

    if (index < 0 && index > this.size) {
      throw new IllegalArgumentException("Add failed. Require index > 0 and index <= size. ")
    }

    for (int i = size - 1; i >= index; i--) {
      this.data[i + 1] = this.data[i];
    }

    this.data[index] = e;
    size++;
  }

}

```
