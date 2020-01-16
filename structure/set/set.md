### 集合(Set)

#### 集合特性
  * 元素不重复

#### 利用二分搜索树实现集合

更具上面的特性, 集合中是不包含重复元素的, 而我们上一章(bst)学习的二分搜索树恰好就是一个不包含重复元素的树结构  
我们可以利用二分搜索树作为集合的底层实现结构。

我们先来看看集合接口的几个方法把。

```java
public interface Set<E> {
    void add(E e); // 添加元素
    void remove(E e); // 删除元素
    boolean contains(E e); // 是否包含元素
    int getSize(); // 集合大小
    boolean isEmpty(); // 集合是否为空
}
```

创建一个实现该接口的类

```java


/**
 * 基于二分搜索树的集合类
 */
public class BSTSet<E extends Comparable<E>> implements Set<E> {
    private BST<E> bst;

    public BSTSet() {
        this.bst = new BST<>();
    }

    @Override
    public void add(E e) {
        bst.add(e);
    }

    @Override
    public void remove(E e) {
        bst.remove(e);
    }

    @Override
    public boolean contains(E e) {
        return bst.contains(e);
    }

    @Override
    public int getSize() {
        return bst.size();
    }

    @Override
    public boolean isEmpty() {
        return bst.isEmpty();
    }
}
```

这里可以看到, 所有的操作都是通过二分搜索树来实现的。


通过下面程序, 进行一个简单测试

```java

public class BSTSetTest {
    public static void main(String[] args) {
        Set<Integer> set = new BSTSet<>();
        set.add(1);
        set.add(2);
        set.add(3);
        set.add(4);
        set.add(5);
        set.add(1);
        set.add(1);
        set.add(1);
        set.add(2);
        System.out.println(set.getSize());

        set.remove(2);
        System.out.println(set.getSize());

        System.out.println(set.contains(11));
    }
}
```



#### 基于链表实现集合

这里使用的是我们在学习链表时候的累, 但是需要扩展一个方法, 之前没有删除元素的方法, 现在新增一个.

```java

// linkedlist.java
// 从链表中删除元素
public void removeElement(E e) {
    Node prev = dummyHead;
    while (prev.next != null) {
        if (prev.next.e.equals(e)) {
            break;
        }

        prev = prev.next;
    }

    if (prev.next != null) {
        Node delNode = prev.next;
        prev.next = delNode.next;
        delNode.next = null;
        size -- ;
    }
}
```


```java
public class LinkedListSet<E> implements Set<E> {

    private LinkedList<E> list;

    public LinkedListSet() {
        this.list = new LinkedList<>();
    }

    @Override
    public void add(E e) {
        if (!list.contains(e)) {
            // 如果不包含元素则进行添加
            list.addFirst(e);
        }
    }

    @Override
    public void remove(E e) {
        list.removeElement(e);
    }

    @Override
    public boolean contains(E e) {
        return list.contains(e);
    }

    @Override
    public int getSize() {
        return list.getSize();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }
}
```

测试方法
```java
public static void main(String[] args) {
        Set<Integer> set = new LinkedListSet<>();
        set.add(1);
        set.add(2);
        set.add(3);
        set.add(2);
        set.add(1);
        set.add(3);
        set.add(3);
        set.add(3);
        System.out.println(set.getSize());

        set.remove(2);
        System.out.println(set.getSize());

}
```
