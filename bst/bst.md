### 二分搜索树(Binary Search Tree)

#### 为什么要使用树?
比如说我们电脑有磁盘, 磁盘下面有很多文件夹, 每个文件夹都分门别类的存放自己要查找的东西。
假设有文学类文件夹、编程开发文件夹、画画文件夹等等等。

文学文件夹下又有散文、诗歌、小说、童话等等<br />
编程文件夹下又有C++、JAVA、Python等等、<br />
画画文件夹下又有油画、插画等等 <br />

每个大类下又分各种小类, 直到不能再细分到一个领域了。如果没有树结构的话, 我们如何能在大量文件中
查找到我们想要的书呢？即使能查到, 效率也是非常低的。


#### 二叉树
在了解二分搜索树之前, 我们先来看看二叉树长什么样子。

通过下图, 我们来大概了解一下什么是二叉树。

![avatar](https://github.com/basebase/img_server/blob/master/common/bst01.png?raw=true)

二叉树和链表一样属于动态数据结构。我们不需要在创建数据结构的时候, 就去决定这个数据结构能够存储多少元素的问题。
如果要添加元素, 就new一个新空间添加到数据结构中, 删除也是一样的。

更具上图, 我们如何构建一个二叉树?
```java
class Node {
  E e ;
  Node left ; // 左孩子
  Node right ; // 右孩子
}
```

二叉树居右唯一一个根节点就是28这个元素。

在创建节点的同时, 我们还可以指定我们左边和右边的孩子是谁, 比如上图中 <br />
```text
元素28的左孩子是16右孩子是30
元素16的左孩子是13右孩子是22
元素30的左孩子是29右孩子是42
```

每个节点都有一个父亲节点, 除了根节点没有父节点外。
```text
16的父亲节点是28
30的父亲节点是28
```

二叉树顾名思义就是, 每个节点最多只能分2个节点, 如果有多个节点我们可以更具它分为几个叉就称为几叉树(多叉树)。

如果一个孩子都没有的我们称为叶子节点(左右孩子都为空就是叶子节点)。



##### 二叉树的递归

二叉树具有天然的递归性, 每个节点又可以看做是一个二叉树。

![avatar](https://github.com/basebase/img_server/blob/master/common/bst02.png?raw=true)


##### 二叉树一些形态

上面的话, 我们都是满二叉树, 但是很多时候都不是。如下图

<strong>只有一个节点或者为空的二叉树</strong>
![avatar](https://github.com/basebase/img_server/blob/master/common/bst07.png?raw=true)

<strong>只有左子树的二叉树</strong>
![avatar](https://github.com/basebase/img_server/blob/master/common/bst06.png?raw=true)

![avatar](https://github.com/basebase/img_server/blob/master/common/bst03.png?raw=true)

![avatar](https://github.com/basebase/img_server/blob/master/common/bst04.png?raw=true)

![avatar](https://github.com/basebase/img_server/blob/master/common/bst05.png?raw=true)





#### 二分搜索树

定义:
  * 若任意节点的左子树不为空, 则左子树上所有及诶按的值均小于它的根节点值
  * 若任意节点的右子树不为空, 则右子树上所有节点的值均大于它的更及诶按的值
  * 任意节点的左、右子树分别为二分搜索树


![avatar](https://github.com/basebase/img_server/blob/master/common/bst08.png?raw=true)


二叉树中每个元素都需要进行比较, 而且并不是都是一个满二叉树

![avatar](https://github.com/basebase/img_server/blob/master/common/bst11.png?raw=true)



#### 实战部分
经过前面的学习, 我们已经大概清楚什么是二分搜索树了。下面我们通过代码来实现把。



```java

/****
 *
 * 存储的元素需要有可比较性, 所以我们需要继承Comparable
 * @param <E>
 */
public class BST<E extends Comparable<E>> {

    private class Node {
        E e ;
        Node left ;
        Node right ;

        public Node(E e) {
            this.e = e;
            left = null ;
            right = null ;
        }
    }

    private Node root ;
    private int size ;

    public BST() {
        this.root = null;
    }

    public int size() {
        return size ;
    }

    public boolean isEmpty() {
        return size == 0 ;
    }
}
```
