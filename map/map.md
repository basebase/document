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
