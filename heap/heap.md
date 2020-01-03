#### Heap(堆)


#### Heap学前知识

堆的概念:  
  N个元素序列[k1, k2, k3, k4, k5, k6...kn]当且仅当满足以下关系时才会被称为堆。  
  ```text
  当数据下标为1时: ki <= k2i, ki <= k2i+1 或者 ki >= k2i, ki >= k2i + 1
  当数据下标为0时: ki <= k2i + 1, ki <= k2i + 2 或者 ki >= k2i + 1, ki >= k2i + 2
  ```
  堆(heap)的实现通常是通过构造二叉堆, 因为应用较为普遍, 当不加限定时, 堆通常指的就是二叉堆。


二叉堆:
+ 二叉堆是一棵完全二叉树(参考图1-1)
+ 堆中的节点值总是不大于其父亲节点的值, 这种我们一般称为最大堆。反之亦然我们称为最小堆。(参考图1-2)
+ 利用数组实现二叉堆(参考图1-3)
  + 使用下标0的公式:
    ```text
      parent(i) = i / 2
      left child (i) = 2 * i
      right child (i) = 2 * i + 1
    ```
  + 使用下标1的公式:
    ```text
      parent(i) = (i - 1) / 2
      left child (i) = 2 * i + 1
      right child (i) = 2 * i + 2
    ```


图1-1
![avatar](https://github.com/basebase/img_server/blob/master/common/heap01.png?raw=true)
<br /><br />

图1-2
![avatar](https://github.com/basebase/img_server/blob/master/common/heap02.png?raw=true)
<br /><br />

图1-3
![avatar](https://github.com/basebase/img_server/blob/master/common/heap03.png?raw=true)
<br /><br />


#### 最大堆的设计实现

##### 初步结构

由于堆中的元素是需要进行比较的, 所以插入进来的元素都是需要带有可比较性的。这里我们继承Comparable即可。
这里我们用的是java自带的动态数组, 这样避免空间不充足问题。

为了方便查找元素的父节点以及左右孩子节点, 我们将其封装成方法, 这样无论是从数组下标0或者下标1开始对于我们来说
都是不关的, 我们只关心返回正确的节点索引位置数据。

```java

public class MaxHeap<E extends Comparable<E>> {

    private ArrayList<E> array;

    public MaxHeap(int capacity) {
        this.array = new ArrayList<>(capacity);
    }

    public MaxHeap() {
        this.array = new ArrayList<>();
    }

    public int size() {
        return array.size();
    }

    public boolean isEmpty() {
        return array.size() == 0;
    }

    // 返回父元素在二叉堆中数组的索引位置
    private int parent(int index) {
        if (index == 0)
            throw new IllegalArgumentException("该索引没有父节点");
        return (index - 1) / 2 ;
    }

    // 返回左孩子索引
    private int leftChild(int index) {
        return index * 2 + 1;
    }

    // 返回孩子索引
    private int rightChild(int index) {
        return index * 2 + 2;
    }

}
```
