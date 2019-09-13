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


public class Array {

    private static final int DEFAULT_CAPACITY = 10;

    private int[] data; // 数组容量
    private int size; // 存放了多少元素


    /***
     * 提供初始化大小的数组
     * @param capacity
     */
    public Array(int capacity) {
        this.data = new int[capacity];
        this.size = 0;
    }

    /**
     * 提供默认初始化大小数组
     */
    public Array() {
        this(DEFAULT_CAPACITY);
    }

    /**
     * @return 实际元素个数
     */
    public int getSize() {
        return size;
    }

    /***
     *
     * @return 数组容量
     */
    public int getCapacity() {
        return data.length;
    }

    /**
     * 判断数组是否为空
     * @return
     */
    public boolean isEmpty() {
        return size == 0 ;
    }

    /**
     * 向数组头部添加元素
     * @param e
     */
    public void addFirst(int e) {
        add(0, e);
    }

    /**
     * 向数组末尾添加元素
     * @param e 实际值
     */
    public void addLast(int e) {
//        if (size >= data.length) {
//            throw new IllegalArgumentException("数组已满.");
//        }
//
//        // 把元素添加到数组末尾
//        data[size] = e;
//        // 实际元素加1, 否则数组一直覆盖当前下标的值
//        size++;
        add(size, e);
    }

    /***
     * 向数组指定位置添加元素
     * @param index 指定下标的位置
     * @param e 元素值
     */
    public void add(int index, int e) {

        if (size >= data.length) {
            throw new IllegalArgumentException("数组已满.");
        }

        if (index < 0 || index > size) {
            throw new IllegalArgumentException("请输入正确的下标位置");
        }

        // 将数组的元素向后面移动
        for (int i = size; i > index; i--) {
            data[i] = data[i - 1];
        }

        // 元素移动完毕后, 直接指定对应的下标值赋值
        data[index] = e;
        size ++;
    }


    /**
     * 删除数组头部
     * @return
     */
    public int removeFirst() {
        return remove(0);
    }

    /***
     * 删除数组尾部
     * @return
     */
    public int removeLast() {
        return remove(size - 1);
    }

    /**
     * 删除指定元素
     * @param e
     */
    public void removeElement(int e) {
        int removeElementIndex = find(e);
        remove(removeElementIndex);
    }

    /***
     * 删除指定位置元素
     * @param index
     * @return
     */
    public int remove(int index) {
        if (index < 0 || index >= size) {
            throw new IllegalArgumentException("请输入正确的下标值");
        }

        int removeElement = get(index);
        // 把后面的元素值写入到前一个元素位置
        for (int i = index; i < size - 1; i++) {
            data[i] = data[i + 1];
        }

        size --;
        return removeElement;
    }

    /***
     * 获取index索引位置的元素
     * @param index
     * @return
     */
    public int get(int index) {
        if (index < 0 || index >= size) {
            throw new IllegalArgumentException("输入正确的下标");
        }

        return data[index];
    }


    /***
     * 修改index元素的值为e
     * @param index
     * @param e
     */
    public void set(int index, int e) {
        if (index < 0 || index >= size) {
            throw new IllegalArgumentException("输入正确的下标");
        }

        data[index] = e;
    }

    /**
     * 判断数组是否包含元素
     * @param e
     * @return
     */
    public boolean contains(int e) {
        for (int i = 0; i < size; i++) {
            if (data[i] == e) {
                return true;
            }
        }

        return false;
    }

    /**
     * 查找元素, 如果找到元素返回对应的下标, 否则返回-1
     * @param e
     * @return
     */
    public int find(int e) {
        for (int i = 0 ; i < size; i ++) {
            if (data[i] == e) {
                return i;
            }
        }

        return -1;
    }


    @Override
    public String toString() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("[");
        for (int i = 0; i < size; i ++) {
            if (i < size - 1) {
                buffer.append(data[i]).append(",");
            } else {
                buffer.append(data[i]);
            }
        }

        buffer.append("]");

        return String.format("数组的元素为: %s 数组容量为: %s\n%s", size, getCapacity(), buffer.toString());
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


我们更新使用泛型后, Array类为

```java


public class Array<E> {

    private static final int DEFAULT_CAPACITY = 10;

    private E[] data; // 数组容量
    private int size; // 存放了多少元素


    /***
     * 提供初始化大小的数组
     * @param capacity
     */
    public Array(int capacity) {
        // 需要注意的就是, 泛型数组需要先用Object创建然后强转为泛型
        this.data = (E[]) new Object[capacity];
        this.size = 0;
    }

    /**
     * 提供默认初始化大小数组
     */
    public Array() {
        this(DEFAULT_CAPACITY);
    }

    /**
     * @return 实际元素个数
     */
    public int getSize() {
        return size;
    }

    /***
     *
     * @return 数组容量
     */
    public int getCapacity() {
        return data.length;
    }

    /**
     * 判断数组是否为空
     * @return
     */
    public boolean isEmpty() {
        return size == 0 ;
    }

    /**
     * 向数组头部添加元素
     * @param e
     */
    public void addFirst(E e) {
        add(0, e);
    }

    /**
     * 向数组末尾添加元素
     * @param e 实际值
     */
    public void addLast(E e) {
//        if (size >= data.length) {
//            throw new IllegalArgumentException("数组已满.");
//        }
//
//        // 把元素添加到数组末尾
//        data[size] = e;
//        // 实际元素加1, 否则数组一直覆盖当前下标的值
//        size++;
        add(size, e);
    }

    /***
     * 向数组指定位置添加元素
     * @param index 指定下标的位置
     * @param e 元素值
     */
    public void add(int index, E e) {

        if (size >= data.length) {
            throw new IllegalArgumentException("数组已满.");
        }

        if (index < 0 || index > size) {
            throw new IllegalArgumentException("请输入正确的下标位置");
        }

        // 将数组的元素向后面移动
        for (int i = size; i > index; i--) {
            data[i] = data[i - 1];
        }

        // 元素移动完毕后, 直接指定对应的下标值赋值
        data[index] = e;
        size ++;
    }


    /**
     * 删除数组头部
     * @return
     */
    public E removeFirst() {
        return remove(0);
    }

    /***
     * 删除数组尾部
     * @return
     */
    public E removeLast() {
        return remove(size - 1);
    }

    /**
     * 删除指定元素
     * @param e
     */
    public void removeElement(E e) {
        int removeElementIndex = find(e);
        remove(removeElementIndex);
    }

    /***
     * 删除指定位置元素
     * @param index
     * @return
     */
    public E remove(int index) {
        if (index < 0 || index >= size) {
            throw new IllegalArgumentException("请输入正确的下标值");
        }

        E removeElement = get(index);
        // 把后面的元素值写入到前一个元素位置
        for (int i = index; i < size - 1; i++) {
            data[i] = data[i + 1];
        }

        size --;
        return removeElement;
    }

    /***
     * 获取index索引位置的元素
     * @param index
     * @return
     */
    public E get(int index) {
        if (index < 0 || index >= size) {
            throw new IllegalArgumentException("输入正确的下标");
        }

        return data[index];
    }


    /***
     * 修改index元素的值为e
     * @param index
     * @param e
     */
    public void set(int index, E e) {
        if (index < 0 || index >= size) {
            throw new IllegalArgumentException("输入正确的下标");
        }

        data[index] = e;
    }

    /**
     * 判断数组是否包含元素
     * @param e
     * @return
     */
    public boolean contains(E e) {
        for (int i = 0; i < size; i++) {
            if (data[i].equals(e)) {
                return true;
            }
        }

        return false;
    }

    /**
     * 查找元素, 如果找到元素返回对应的下标, 否则返回-1
     * @param e
     * @return
     */
    public int find(E e) {
        for (int i = 0 ; i < size; i ++) {
            if (data[i].equals(e)) {
                return i;
            }
        }

        return -1;
    }


    @Override
    public String toString() {
        StringBuffer buffer = new StringBuffer();
        buffer.append("[");
        for (int i = 0; i < size; i ++) {
            if (i < size - 1) {
                buffer.append(data[i]).append(",");
            } else {
                buffer.append(data[i]);
            }
        }

        buffer.append("]");

        return String.format("数组的元素为: %s 数组容量为: %s\n%s", size, getCapacity(), buffer.toString());
    }
}

```


#### 动态数组

先通过一张图片来了解一下, 数组是如何动态扩容的。

![avatar](https://github.com/basebase/img_server/blob/master/common/array06.png?raw=true)

```java

public void add(int index, E e) {
  if (size >= data.length) {
//            throw new IllegalArgumentException("数组已满.");
            // 动态扩容数组
            resize(data.length * 2);
        }
}

public E remove(int index) {

  // 如果当前元素个数小到当前数组的一半, 整个数组只有一半被利用, 将数组容量缩小
  if (size == data.length / 2) {
    resize(data.length / 2);
  }
}

/***
 * 更新数组大小
 * @param newCapacity
 */
private void resize(int newCapacity) {
    E[] newData = (E[]) new Object[newCapacity];
    for (int i = 0; i < size; i ++) {
        newData[i] = data[i];
    }

    data = newData;
}
```
