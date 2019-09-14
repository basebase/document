### 队列

#### 基本介绍
队列是一个先进先出(FIFO, First-In-First-Out)的线性表, 队列只允许在后端(rear)进行插入操作, 在前端(front)进行删除操作。

#### 特点
  * 先进先出

队列操作如下图所示

![avatar](https://github.com/basebase/img_server/blob/master/common/queue01.jpeg?raw=true)



#### 设计一个Queue
  * 入队
  * 出队
  * 队首
  * 队列长度
  * 队列是否为空


```java

public interface Queue<E> {
    // 入队
    void enqueue(E e);
    // 出队
    E dequeue();
    // 获取队首
    E getFront();
    int getSize();
    boolean isEmpty();
}
```


```java



public class ArrayQueue<E> implements Queue<E> {

    private Array<E> array;

    public ArrayQueue(int capacity) {
        this.array = new Array<>(capacity);
    }

    public ArrayQueue() {
        this.array = new Array<>();
    }

    @Override
    public void enqueue(E e) {
        array.addLast(e);
    }

    @Override
    public E dequeue() {
        return array.removeFirst();
    }

    @Override
    public E getFront() {
        return array.getFirst();
    }

    @Override
    public int getSize() {
        return array.getSize();
    }

    @Override
    public boolean isEmpty() {
        return array.isEmpty();
    }

    @Override
    public String toString() {
        StringBuffer buff = new StringBuffer();
        buff.append("Queue: front [");
        for (int i = 0; i < array.getSize(); i ++) {
            buff.append(array.get(i));
            if (i < array.getSize() - 1)
                buff.append(",");
        }

        buff.append("] tail");
        return buff.toString();
    }
}
```
