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


 上面, 我们已经实现了在链表头部添加元素, 那么现在我们指定位置添加元素如何处理呢？

 需要注意的是: 如果指定的是链表头, 由于头部没有上一个节点, 所以需要单独处理一下。
 后面通过虚拟头结点来解决。

 1. 对于这个链表, 要在这个链表索引(链表是无索引的, 这里只是借用索引这个概念来阐述)为2的地方添加一个新的元素666

 2. 首先遍历找到索引为2的前一个节点prev
 3. 然后prev.next指向存放为2的元素节点, 同时存放666节点, node.next也指向它, 因此得到node.next = prev.next(将prev.next赋值给node.next)
 4. 之后存放666节点, 即prev.next = node(将node赋值给prev.next)



 ![avatar](https://github.com/basebase/img_server/blob/master/common/linkedlist3.jpg?raw=true)



 ```java

 // 在链表末尾添加元素
     public void addLast(E e) {
         add(size, e);
     }

     // 在链表的index(0-based)位置添加元素
     public void add(int index, E e) {Â
         if (index > size || index < 0) {
             throw new IllegalArgumentException("添加失败, 请输入正确的索引位置");
         }

         if (index == 0) {
             addFirst(e);
         } else {
             Node prev = head;
             for (int  i = 0 ; i < index - 1; i ++) {
                 prev = prev.next;
             }

 //            Node node = new Node(e);
 //            node.next = prev.next;
 //            prev.next = node;

             prev.next = new Node(e, prev.next);
             size ++;
         }
     }
 ```


 ### 虚拟头结点
 在向链表添加元素的时候, 我们遇到了一个问题, 在链表头添加元素和其它位置添加元素逻辑上会有区别。
 为什么在为链表头添加元素会有不同: 这是因为要找到待添加元素之前的位置, 由于链表头没有钱一个节点, 所以在逻辑上会特殊一些。

 这也就是衍生出了"虚拟头节点"。


 ![avatar](https://github.com/basebase/img_server/blob/master/common/linkedlist4.png?raw=true)

 其中, 虚拟头结点中是不包含任何数据的.

更新后的代码

```java

// - private Node head ;
// +
private Node dummyHead ; // 虚拟头结点

public LinkedList() {
       this.size = 0;
       // +
       this.dummyHead = new Node(null, null); // 虚拟头结点, 里面不存放内容
// -       this.head = null;
}

public void add(int index, E e) {
        if (index < 0 || index > size) {
            throw new IllegalArgumentException("请传入正确的位置.");
        }

        Node prev = dummyHead;
        for (int i = 0 ; i < index /*index - 1*/; i ++) {
            prev = prev.next;
        }

        prev.next = new Node(e, prev.next);
    }
```

现在, 我们在从位置0上添加元素5就会发现

prev = dummyHead

而i < index 吗？不小于则不进行next,所以dummyHead的next是0

我们从新创建一个元素5, 他的下一个next就是0, 然后从新赋值给prev.next即dummyHead.next = 新元素数据
