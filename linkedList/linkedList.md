### 链表

#### 链表介绍
前面我们学习了[数组, 栈, 队列]这些数据结构都属于线性结构, 链表也是线性结构, 前面的栈和队列我们底层都是使用的数组虽然是说动态扩展, 但也是通过resize()函数来实现的, 但是链表却是货真价实的动态的结构, 我们不需要为期初始化容量, 也不需要去关心扩容。链表只需要知道你的下一个节点是谁即可。


#### 链表的优缺点
  * 优点
    * 动态

  * 缺点
    * 不支持索引查询(无法像数组一样通过下标获取数据)

链表常见的有
  * 单向链表
  * 双向链表
  * 循环链表


#### 单向链表
链表中最简单的一种是单向链表, 它包含两个域, 一个信息域和一个指针域。
这个链接指向列表中的下一个及诶按, 而最后一个节点指向空值。

 ![avatar](https://github.com/basebase/img_server/blob/master/common/linklist01.png?raw=true)
一个单链表包含两个值: 当前节点的值和一个指向下一个节点的链接

一个单向链表的节点被分为两部分。第一个部分保存或显示关于节点信息, 第二个部分存储下一个节点的地址。单向链表只可向一个方向遍历。

链表最基本的结构是在每个节点保存数据和到下一个节点的地址, 在最后一个节点保存一个特殊的结束标记, 另外在一个固定的位置保存指向第一个节点的指针, 有时候也会同事存储指向最后一个节点的指针. 一般查找一个节点的时候需要从第一个节点开始每次访问下一个节点, 一直访问到需要的位置。但是也可以提前把一个节点的位置另外保存起来, 然后直接访问。
当然如果只是访问数据就没有必要了, 不如在链表上存储指向实际数据的指针。这样一般是为了访问链表中的下一个或者前一个(双向链表)节点。





#### 单向链表例子


```java

public class LinkedList<E> {

    /**
      创建一个节点
    */
    private class Node {
        private E e;
        private Node next;

        public Node(E e, Node next) {
            this.e = e;
            this.next = next;
        }

        public Node(E e) {
            this(e, null);
        }

        public Node() {
            this(null, null);
        }

        @Override
        public String toString() {
            return e.toString();
        }
    }
}
```


#### 如何向链表添加元素

我们最简单的添加方式就是在头部添加, 稍微麻烦一点的话就在指定位置前添加。
因此, 我们有两种添加方式

  * 头部添加元素
  * 指定位置添加元素



我们先来看看, 如何在头部添加元素, 头部添加元素也是最简单的。

1. 假设要将666这个元素添加到链表中。
2. 相应的需要在Node节点里存放666这个元素, 以及相应的next(指向)
3. 然后node节点的next指向链表的头, 即node.next = head(将head赋值给node.next)
4.最后head也指向存放666的node节点, 即head = node

 ![avatar](https://github.com/basebase/img_server/blob/master/common/linkedlist2.jpg?raw=true)


 ```java

     private int size;
     private Node head;

     public LinkedList() {
         this.size = 0;
         this.head = null;
     }

     // 获取链表元素个数
     public int getSize() {
         return size;
     }

     // 判断链表是否为空
     public boolean isEmpty() {
         return size == 0;
     }


     // 在链表头添加新的元素
     public void addFirst(E e) {


 //        Node node = new Node(e);
 //        node.next = head;
 //        head = node;

         // 这一句话等价上面三句
         head = new Node(e, head);

         size ++;
     }
 ```
