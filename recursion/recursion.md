### 链表与递归

#### 简单介绍
前面我们已经介绍了链表, 然而这个主题主要是用于扩展学习链表的, 以及递归的基本使用。
链表练习主要以LeetCode为主, 然后给大家认识一下基础的递归使用以及调用流程等。


#### LeetCode链表问题

##### 203. 移除链表元素

```Text
删除链表中等于给定值 val 的所有节点
示例:
输入: 1->2->6->3->4->5->6, val = 6 <>
输出: 1->2->3->4->5
```
然后, LeetCode给出的Node代码如下:

```java
public class ListNode {
  int val;
  ListNode next;
  ListNode(int x) { val = x; }
}
```

使用非虚拟头结点解决方法:

```java

public class Solution {
    public ListNode removeElements(ListNode head, int val) {

        // 删除头结点, 需要优先判断是否为空
        while (head != null && head.val == val) {
            head = head.next;
        }

        if (head == null) {
            return null; // 如果全部都是要删除的节点数据, 则中间数据不需要处理了
        }

        ListNode prev = head ;
        // 处理中间数据集
        while (prev.next != null) {
            if (prev.next.val == val) {
                prev.next = prev.next.next;
            } else {
                prev = prev.next;
            }
        }

        return head;
    }

    public static void main(String[] args) {
        Solution s = new Solution();
        ListNode node1 = new ListNode(3);
        ListNode node2 = new ListNode(3);
        ListNode node3 = new ListNode(3);
        ListNode node4 = new ListNode(3);

        node1.next = node2;
        node2.next = node3;
        node3.next = node4;

        s.removeElements(node1, 3);
    }
}
```


虚拟头结点版本
```java
public ListNode removeElements(ListNode head, int val) {

        // 由于虚拟节点永远不会被访问到, 所以给个负数
        ListNode dummyHead = new ListNode(-1);
        dummyHead.next = head;


        // 应为有了虚拟头结点, 所以不需要关心头节点问题, 直接遍历
        ListNode prev = dummyHead ;
        // 处理中间数据集

        while (prev.next != null) {
            if (prev.next.val == val) {
                prev.next = prev.next.next;
            } else {
                prev = prev.next;
            }
        }

        // 虚拟节点是不会外暴露的, 需要需要next
        return dummyHead.next;
}
```
