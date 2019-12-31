### 映射(Map)

#### 映射是什么?
如果使用白话来说的话, 就是更具特定唯一的信息来找到对应的实体。  
比如说我们查字典, 要查询的字能找到对应的解释。
比如我们的有身份证id就能查到对应的人信息。  
比如我们有快递id就能知道当前快递在什么位置  
等等, 都是通过映射的关系来进行实现。


#### 映射结构特点
  * 以K,V键值对存储的数据
  * 根据K能快速的寻找到V



##### 映射接口的设计
我们优先创建一个父类接口, 并设计两个子类一个是链表来实现, 一个是二分搜索树来实现。

```java

public interface Map<K, V> {
    // 添加数据
    void add(K key, V value);
    // 删除数据
    V remove(K key);
    // 判断K是否存在
    boolean contains(K key);
    // 更具K获取value
    V get(K key);
    // 设置新的value
    void set(K key, V newValue);
    int getSize();
    boolean isEmpty();
}
```


#### 基于链表实现映射数据结构

学习集合的时候, 我们也使用了链表进行集合的设计, 但是在设计映射对象的时候可能就无法满足了, 毕竟要K和V一对。
所以我们需要重新构建一下Node对象。并重新设计add和remove等方法。

```java


public class LinkedListMap<K, V> implements Map<K, V> {

    // 之前写的链表只有一个元素是无法满足map的结构, 所以重新定义一下Node
    private class Node {
        public K key;
        public V value;
        public Node next;

        public Node() {
            this(null, null, null);
        }

        public Node(K key, V value, Node next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        @Override
        public String toString() {
            return key + " : " + value;
        }
    }


    private int size ;
    private Node dummyHead ; // 虚拟头结点

    public LinkedListMap() {
        dummyHead = new Node();
        size = 0;
    }

    /**
     * 更具K获取到具体的Node对象
     * @param key
     * @return
     */
    private Node getNode(K key) {
        Node cur = dummyHead.next; // 实际的头节点
        while (cur != null) {
            if (cur.key.equals(key)) {
                return cur ;
            }

            cur = cur.next;
        }

        return null;
    }

    @Override
    public void add(K key, V value) {
        Node cur = getNode(key);
        if (cur == null) {
            dummyHead.next = new Node(key, value, dummyHead.next);
            size ++ ;
        } else {
            // 已经存在了, 这里的话不抛出异常但是给个提示吧
            System.out.println("添加的key="+key+"已存在, 给你更新啦!");
            cur.value = value;
        }
    }

    @Override
    public void set(K key, V newValue) {
        Node cur = getNode(key);
        if (cur == null) {
            throw new IllegalArgumentException("当前key="+key+"不存在, 请检查是否拼写错误");
        }

        cur.value = newValue;
    }

    @Override
    public V remove(K key) {

        Node prev = dummyHead.next;
        while (prev.next != null) {
            if (prev.next.key.equals(key)) {
                break;
            }

            prev = prev.next;
        }

        if (prev.next != null) {
            Node delNode = prev.next;
            prev.next = delNode.next;
            delNode.next = null;
            size -- ;
            return delNode.value;
        }

        return null;
    }

    @Override
    public boolean contains(K key) {
        return getNode(key) != null;
    }

    @Override
    public V get(K key) {
        Node cur = getNode(key);
        return cur == null ? null : cur.value;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        return size == 0;
    }
}
```


接下来测试一下我们基于链表的映射数据结构把。

```java

public static void main(String[] args) {
        String[] words = {"A", "B", "C", "D", "E", "A", "A", "A", "B", "B", "C", "C", "F", "F", "F", "K",};
        Map<String, Integer> map = new LinkedListMap<>();
        for (String word : words) {
            if (map.contains(word)) {
                Integer v = map.get(word) + 1;
                map.set(word, v);
            } else {
                map.add(word, 1);
            }
        }

        System.out.println("总共: " + map.getSize());

        System.out.println(map.get("A"));
        System.out.println(map.get("B"));
        System.out.println(map.get("C"));
        System.out.println(map.get("D"));
        System.out.println(map.get("E"));
        System.out.println(map.get("F"));
        System.out.println(map.get("K"));
}
```
