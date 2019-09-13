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
  * 数组的删除和新增都需要移动元素


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


用一张图片来表示一下数组把。


![avatar](https://github.com/basebase/img_server/blob/master/common/array01.png?raw=true)


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



我们来看看怎么设计这样的一个数组?

![avatar](https://github.com/basebase/img_server/blob/master/common/array02.png?raw=true)

数组有存储实际元素的大小, 有一个数组的总大小, 我们可以对数组进行增删改查操作。
当存入一个实际元素的的时候size就加1, 当删除一个元素的时候size就减1.



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

  /**
    获取index索引位置数据
  */
  public int get(int index) {
    if (index < 0 || index < size) {
      throw new IllegalArgumentException("Get failed. Inex is illegal.")
    }

    return data[index];
  }

  /**
    修改index索引位置的值为e
  */
  public void set(int index, int e) {
    if (index < 0 || index < size) {
      throw new IllegalArgumentException("Get failed. Inex is illegal.")
    }

    data[index] = e;
  }

}

```


向数组末尾添加元素流程如下图：

![avatar](https://github.com/basebase/img_server/blob/master/common/array03.jpg?raw=true)


向数组任意位置添加元素流程如下图:

![avatar](https://github.com/basebase/img_server/blob/master/common/array04.jpg?raw=true)


向数组任意位置删除元素流程如下图:

![avatar](https://github.com/basebase/img_server/blob/master/common/array05.jpg?raw=true)





#### 数组泛型

使用泛型，可以让我们的数组放入“任意”数据类型。
但是不可以放基本数据类型, 只能放类对象, java中基本类型有8种

**boolean, byte, char, short, int, long, float, double**

那我们如果要放基本类型怎么实现呢?
别着急JDK给我们做了包装类, 分别对应上面8种的基本类型

**Boolean, Byte, Char, Short, Integer, Long, Float, Double**

这样, 我们就可以使用了。
