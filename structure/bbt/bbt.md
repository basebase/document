#### 平衡二叉树(Balanced Binary Tree)


##### 什么是平衡二叉树?

###### 平衡二叉树的基本概念

在介绍平衡二叉树之前, 我们先来回忆一下二分搜索树的一个问题。
假设有一组数[1, 2, 3, 4, 5, 6]如果我们以顺序添加到二分搜索树中, 那么这颗二分搜索树就会退化成一个链表(如图[1-1]展示)。这就大大降低二分搜索树的效率。那么怎么解决这个问题呢? 我们需要在现有的二分搜索树的基础上添加一定的机制, 使得我们的二分搜索树能够维持平衡二叉树。

[图1-1 [退化成链表的二分搜索树]]
![1-1](https://github.com/basebase/img_server/blob/master/common/bbt01.png?raw=true)


那么平衡二叉树是什么? 在我们之前的树结构中有遇到过平衡二叉树吗?


1. 一颗满二叉树一定是一颗平衡二叉树。

2. 完全二叉树(堆)。对于完全二叉树来说, 空缺的节点部分一定是在树的右下部分, 相应的对于一颗完全二叉树整棵树的叶子节点最大的深度值和最小的深度值相差不会超过1。也就是说我们所有的叶子节点要么在最后一层, 要么在倒数第二层。

3. 线段树也就是一种平衡二叉树。虽然线段树不是一个完全二叉树, 对于线段树来说空出来的部分不一定在整个树的右下角的位置, 但是在一个整体线段树中叶子节点也是在最后一层或者在倒数第二层。对于整棵树来说我们叶子节点的深度相差不会超过1。

以上这些都是平衡二叉树的例子。


平衡二叉树的定义:
+ <strong>对于任意一个节点, 左子树和右子树的高度差不能超过1。</strong>

上面的定义看着和我们的之前的完全二叉树还是线段树这样的二叉树都差不多, 但实际上是有区别的。对于堆和线段树来说, 可以保证任意一个叶子节点相应的高度差都不超过1。而上面的定义是任意一个节点左右子树高度差不超过1。在这个定义下我们得到的平衡二叉树有可能看着不是"那么的"平衡(如图[1-2])。

[图1-2 [一颗平衡二叉树]]
![1-2](https://github.com/basebase/img_server/blob/master/common/bbt02.png?raw=true)

该图中的结构, 显然是不会出现在堆或者线段树这两种树结构中。这棵树看起来稍微有一些偏斜, 但如果仔细去验证每一个节点就会发现, 这棵树是满足平衡二叉树的定义的。

```text
从根节点12开始, 左子树高度是3, 右子树高度是2。高度差为1。没有超过1。
节点8开始, 左子树高度是2, 右子树高度1。高度差为1。没有超过1。
节点18开始, 左子树高度是1, 右子树高度0。高度差为1。没有超过1。
节点5开始, 左子树高度是1, 右子树高度0。高度差为1。没有超过1。

相应的11, 17, 4。这三个节点是叶子节点, 对于叶子节点来说左右子树都是空, 说明左右子树高度都为0, 所以差为0, 也没有超过1。

所以, 这颗树看起来有些偏斜, 但是, 是在我们这个定义下的一颗平衡二叉树。
```

图[1-2]已经是一颗平衡二叉树了, 但是如果我们在这棵树上添加节点的话, 比如添加一个节点2和节点7, 根据二分搜索树的性质, 那么节点2会从根节点一路找下来, 最终添加到节点4左子树中, 相应的如果在添加一个节点7, 节点7会添加到节点5右子树中。就会形成图[1-3]的样子。但是已经不在是一颗平衡二叉树了。

[图1-3 [一颗失去平衡的二叉树]]
![1-3](https://github.com/basebase/img_server/blob/master/common/bbt03.png?raw=true)

可以看到节点8的位置上, 左子树的高度是3, 右子树的高度是1。左右子树的高度差为2。破坏了平衡二叉树的条件。同理根节点12也是一样, 他的左子树高度是4, 右子树高度是2。高度差为2。所以, 现在这棵二叉树不在是一颗平衡二叉树了。

那么如何保持平衡呢? 我们必须保证在插入节点的时候, 相应的也要顾及这颗树的右侧部分。
因为这棵树现在看是向左偏斜的。相应的也要填补这棵树右侧空间的节点。才能继续让这颗树维持平衡二叉树左右子树高度差不超过1这个性质。


##### 节点高度&平衡因子

在具体开发中, 由于要跟踪每一个节点对应的高度是多少, 只有这样才方便我们判断, 当前的二叉树是否是平衡的。所以对之前实现的二分搜索树来说, 要实现平衡二叉树我们只要对每一个节点标注节点高度。这个记录非常的简单。

```text
标注节点高度过程:

对于叶子节点2高度为1, 4这个节点由于有叶子节点2对应的高度为2, 对于叶子节点7高度为1。

对于5这个节点, 由于有左右两颗子树, 左边的子树高度为2, 右边的子树高度为1, 相应的节点5的高度就是左右两颗子树中最高的那棵树在加上1。这个1是节点5自身。所以节点5的高度是3。

叶子节点11的高度为1, 节点8的高度4, 叶子节点17的高度为1。
节点18的高度为2。根节点的高度为5。

这样, 我们就对每一个节点都标注好了高度值。
```

上面, 我们把节点高度值标注好之后, 相应的我们要计算一个"平衡因子"。  
平衡因子: 就是计算左右子树的高度差。计算方法就是[左子树高度-右子树高度]。


```text
平衡因子计算过程:

对于叶子节点2, 它的左右两颗子树相当于是两颗空树, 空树的高度记为0, 相应的叶子节点的平衡因子就是(0 - 0)结果为0。

对于节点4来说左子树高度为1, 右子树为高度0, 即平衡因子为(1 - 0)为1
叶子节点7的平衡因子也为0

节点5的左子树高度为2, 右子树高度为1, 即平衡因子为(2 - 1)为1
叶子节点11的平衡因子也为0

节点8的左子树高度为3, 右子树高度为1, 即平衡因子为(3 - 1)为2, 这就意味着对于8这个节点来说, 左右子树高度差超过1了, 通过这个平衡因子就能看出这棵树已经不是一颗平衡二叉树了。换句话说, 只要平衡因此大于1, 这棵树就不是一颗平衡二叉树了。

叶子节点17的平衡因子也为0
对于节点18来说左子树高度为1, 右子树高度为0, 即平衡因子为(1 - 0)为1

对于根节点12, 左子树高度为4, 右子树高度为2, 即平衡因子为(4 - 2)为2
```

经过上面平衡因子的计算, 我们很清楚的知道有两个节点破坏了平衡二叉树的性质。


[图1-4 [计算二叉树的高度和平衡因子]]
![1-4](https://github.com/basebase/img_server/blob/master/common/bbt04.png?raw=true)


###### 树高度与平衡因子代码实现

我们要实现的平衡树底层还是利用我们之前学习的二分搜索树, 可以沿用之前的代码实现, 所以建议在学习完二分搜索树之后再来阅读本篇。当然如果你已经会了二分搜索树也没关系, 我还是会提供一份完全代码的清单, 哈哈哈~

这里我们只需要关注两个点
  * 节点高度
  * 节点平衡因子

我们在Node类中新增加一个变量"height"来代表当前节点的高度。在构建每一个节点的时候
我们初始化高度都为1, 按照二分搜索树添加的特性, 肯定会一路找下去, 最后肯定是一个叶子节点。

我们添加了两个私有方法, 一个获取高度, 一个计算平衡因子。

那么, 我们在什么时候维护树的高度以及计算平衡因子呢?
当前, 我们在添加节点的时候, 就会计算当前节点的高度以及平衡因子。

```java


public class AVLTree<K extends Comparable<K>, V> {

    private class Node{
        public K key;
        public V value;
        public Node left, right;

        // 树的高度
        public int height;

        public Node(K key, V value){
            this.key = key;
            this.value = value;
            left = null;
            right = null;

            /**
             *  添加元素的时候, 肯定是一个叶子节点, 所以创建默认的高度就是1
             */
            this.height = 1;
        }
    }

    private Node root;
    private int size;

    public AVLTree(){
        root = null;
        size = 0;
    }

    public int getSize(){
        return size;
    }

    public boolean isEmpty(){
        return size == 0;
    }


    /***
     * 获取节点node的高度
     * @param node
     * @return
     */
    private int getHeight(Node node) {
        if (node == null)
            return 0;
        return node.height;
    }

    /***
     * 计算节点node平衡因子
     * @param node
     * @return
     */
    private int getBalanceFactor(Node node) {
        if (node == null)
            return 0;

        /***
         * 平衡因子计算方法:
         *   当前节点左子树高度 - 当前节点右子树高度
         */
        return getHeight(node.left) - getHeight(node.right);
    }

    // 向二分搜索树中添加新的元素(key, value)
    public void add(K key, V value){
        root = add(root, key, value);
    }

    // 向以node为根的二分搜索树中插入元素(key, value)，递归算法
    // 返回插入新节点后二分搜索树的根
    private Node add(Node node, K key, V value){

        if(node == null){
            size ++;
            return new Node(key, value); // 遍历到最后一个节点返回, 默认高度为1
        }

        if(key.compareTo(node.key) < 0)
            node.left = add(node.left, key, value);
        else if(key.compareTo(node.key) > 0)
            node.right = add(node.right, key, value);
        else // key.compareTo(node.key) == 0
            node.value = value;

        /***
         *  在添加元素的时候, 我们需要维护一下树的高度。
         *  如何计算高度呢?
         *    当前节点 + Max(左子树高度, 右子树高度)
         */
        node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

        /***
         *  有了高度之后, 我们很轻易的获取到平衡因子
         */
        int balanceFactor = getBalanceFactor(node);

        // 如果平衡因子大于1, 则破坏了这颗树的平衡性...
        // 这里暂时先不处理, 先输出一段话即可。
        if (Math.abs(balanceFactor) > 1)
            System.out.println("unbalanced : " + balanceFactor);



        return node;
    }

    // 返回以node为根节点的二分搜索树中，key所在的节点
    private Node getNode(Node node, K key){

        if(node == null)
            return null;

        if(key.equals(node.key))
            return node;
        else if(key.compareTo(node.key) < 0)
            return getNode(node.left, key);
        else // if(key.compareTo(node.key) > 0)
            return getNode(node.right, key);
    }

    public boolean contains(K key){
        return getNode(root, key) != null;
    }

    public V get(K key){
        Node node = getNode(root, key);
        return node == null ? null : node.value;
    }

    public void set(K key, V newValue){
        Node node = getNode(root, key);
        if(node == null)
            throw new IllegalArgumentException(key + " doesn't exist!");
        node.value = newValue;
    }

    // 返回以node为根的二分搜索树的最小值所在的节点
    private Node minimum(Node node){
        if(node.left == null)
            return node;
        return minimum(node.left);
    }

    // 删除掉以node为根的二分搜索树中的最小节点
    // 返回删除节点后新的二分搜索树的根
    private Node removeMin(Node node){

        if(node.left == null){
            Node rightNode = node.right;
            node.right = null;
            size --;
            return rightNode;
        }

        node.left = removeMin(node.left);
        return node;
    }

    // 从二分搜索树中删除键为key的节点
    public V remove(K key){

        Node node = getNode(root, key);
        if(node != null){
            root = remove(root, key);
            return node.value;
        }
        return null;
    }

    private Node remove(Node node, K key){

        if( node == null )
            return null;

        if( key.compareTo(node.key) < 0 ){
            node.left = remove(node.left , key);
            return node;
        } else if(key.compareTo(node.key) > 0 ){
            node.right = remove(node.right, key);
            return node;
        } else{   // key.compareTo(node.key) == 0

            // 待删除节点左子树为空的情况
            if(node.left == null){
                Node rightNode = node.right;
                node.right = null;
                size --;
                return rightNode;
            }

            // 待删除节点右子树为空的情况
            if(node.right == null){
                Node leftNode = node.left;
                node.left = null;
                size --;
                return leftNode;
            }

            // 待删除节点左右子树均不为空的情况

            // 找到比待删除节点大的最小节点, 即待删除节点右子树的最小节点
            // 用这个节点顶替待删除节点的位置
            Node successor = minimum(node.right);
            successor.right = removeMin(node.right);
            successor.left = node.left;

            node.left = node.right = null;

            return successor;
        }
    }
}
```


##### 检查二分搜索树性质和平衡性

在介绍AVL树是如何维持自平衡之前, 我们在做一个辅助工作。辅助方法很简单
  * 判断当前树是否为一颗二分搜索树
  * 判断当前树是否为平衡二叉树

对于我们的AVL树来说, 它是对我们的二分搜索树的一个改进。改进的是二分搜索树有可能退化成的链表这种情况。因此引入平衡因子这个概念。AVL同时也是一个二分搜索树。所以也要满足二分搜索树的性质。

<strong>在后续为AVL树添加自平衡机制时, 如果代码有bug, 就很有可能破坏这个性质, 所以设置一个方法用来判断当前AVL树是否还是一颗二分搜索树。</strong>


判断二叉树是否为二分搜索树
```java

/**
 * 判断该二叉树是否是一颗二分搜索树
 * @return
 */
public boolean isBST() {
    if (root == null)
        return true;

    /***
     * 在介绍二分搜索树的时候, 我们介绍过一个特性, 如果是一颗二分搜索树在进行中序遍历它是升序的
     */
    ArrayList<K> keys = new ArrayList<K>();
    inOrder(root, keys);

    for (int i = 1; i < keys.size(); i ++) {
        if (keys.get(i - 1).compareTo(keys.get(i)) > 0) // 如果不是升序的情况则是不是一颗二分搜索树。
            return false;
    }

    return true;
}

private void inOrder(Node node, ArrayList<K> keys) {
    if (node == null)
        return ;

    inOrder(node.left, keys);
    keys.add(node.key);
    inOrder(node.right, keys);
}
```

判断二叉树是否为平衡二叉树
```java
/***
 * 判断该二叉树是否是一颗平衡二叉树。
 * @return
 */
public boolean isBalanced() {
    return isBalanced(root);
}

private boolean isBalanced(Node node) {
    if (node == null)
        return true; // 如果这棵树都为空, 肯定的是一个平衡的 /狗头

    int balanced = getBalanceFactor(node);
    if (Math.abs(balanced) > 1)
        return false;

    return isBalanced(node.left) && isBalanced(node.right); // 左子树和右子树平衡因子都必须在范围1内才是一颗平衡二叉树
}
```



##### 旋转操作基本原理


###### 左旋转和右旋转

AVL是如何实现自平衡的, 在这里主要有两个操作, "左旋转和右旋转"。AVL树是在什么时候维护自平衡的。回忆一下, 我们在二分搜索树插入一个节点的时, 我们需要从根节点一路向下最终寻找到正确的插入位置, 那么, 正确的插入位置都是一个叶子节点。

也就是说, 由于我们新添加了一个节点才有可能导致我们整颗二分搜索树不在满足平衡性。相应的, 这个不平衡的节点只有可能发生在我们插入的位置向父节点去查找, 因为我们是插入了一个节点才破坏了整颗树的平衡性。我们破坏的整棵树的平衡性将反映在这个新的节点的父节点或者祖先节点中。因为在插入这个节点后, 它的父节点或者祖先节点的高度值就需要进行更新。在更新之后平衡因子可能大于1或者小于-1, 也就是左右子树高度差超过了1。

**所以, 我们维护平衡的时机, 应该是, 当我们加入节点后, 沿着节点向上维护平衡性。[参考图2-1]**


[图2-1 [在什么时候维护平衡]]
![2-1](https://github.com/basebase/img_server/blob/master/common/bbt05.png?raw=true)


我们先来看一下不平衡发生的最简单的一种情况[图2-2]中图1的内容。

假设我们现在有一颗空树, 现在我们添加一个节点12, 那么此时这个节点的平衡因子就是0。
然后, 我们有添加一个元素8, 8比12小所以在12的左子树上, 那么节点8的平衡因子就是0, 相应的12这个节点它的平衡因子就更新为1, 然后我们在新增一个节点5, 由于5比8还小, 一路找下来最终成为8的左子树。此时5是一个叶子节点, 它的平衡因子为0, 回到父节点8更新平衡因子为1, 而对于祖先节点12它的平衡因子更新为2。

那么在节点12的位置上, 此时它的平衡因子绝对值大于1, 所以我们需要对它进行一个平衡维护。

再比如说, 我们有 [图2-2]中图2中的情况。如果我们在这颗树上添加一个节点2的话, 这个节点从根节点出发查找, 一直找到节点4并放置在左子树下。

添加完节点2之后, 它是一个新节点它的左右子树都是空, 所以节点2的平衡因子为0。
然后回溯上去到节点2的父节点4, 此时节点4的平衡因子为1, 相应的在往上走对于节点5来说它的平衡因子也为1, 在向上走到节点8它的平衡因子为2, 换句话说, 到节点8的位置打破平衡二叉树的性质。我们需要对节点8进行一个平衡维护。

**我们举的这两个例子, 无论是从空树添加元素还是在已有节点上添加元素, 本质是一样的。"都是插入的元素在不平衡的节点的左侧的左侧", 换句话说, 我们一直在向这棵树的左侧添加元素。最终导致左子树的高度要比右子树的高度要高。与此同时, 我们观察这个不平衡节点的左子树它的平衡因子也是大于0的, 换句话说对于这个不平衡节点它的左孩子这个节点, 也是左子树的高度大于右子树的高度。**

[图2-2 [在什么时候维护平衡]]
![2-2](https://github.com/basebase/img_server/blob/master/common/bbt06.jpg?raw=true)


那么, 上面发生的问题, 我们如何解决呢?  
这里我们通过**右旋转**来进行解决。

这里, 我们将要处理的情况抽象如图[2-3]中第一幅图的样子, 我们有一个Y节点, 对于Y节点来说已经不满足平衡二叉树的条件了。与此同时, 这里我们讨论的是它的左子树的高度要比右子树的高度要高。并且这个高度差是要比1大的。与此同时, 它的左孩子也是同样的情况。左子树的高度是大于等于右子树的。也就是说以Y为根节点这颗子树, 它整体不满足平衡二叉树的性质并且整体是向左倾斜的。

为了不失一般性, 它们的右侧可能也有子树。如图[2-3]中第二幅图的样子, 对于节点Z它的左右是T1和T2。T1和T2可以为空, 只不过不失一般化的处理。让Z也拥有两颗子树。但是Z是一个叶子节点也是完全没问题的, 同理, 对于节点X它右侧可能有子树T3, 对于节点Y可能也有右子树T4。

对于图[2-3]中是Y这个节点左子树过高, 所以希望经过操作后Y这个节点可以保持平衡。与此同时我们整棵子树不能失去二分搜索树的特性。具体如何实现呢?

我们需要进行右旋转, 那么右旋转的过程是怎样的呢?

**首先让X的右子树指向节点Y, 之后我们在让节点Y的左子树指向T3。**
经过上面的操作之后, X成为根节点, 这样一个过程称为右旋转。  
此时, 经过旋转后得到新的二叉树它既满足二分搜索树的性质又满足平衡二叉树的性质。


这里我主要说明一下保持平衡二叉树的性质:
```text
我们将[图2-3]中第2幅图的二叉树转换成第3幅图的二叉树。我们简单分析一下。

图2中我们看到Y是不平衡的节点, 也就意味着Z和X为根的二叉树是平衡二叉树。不然的话, 我们从加入的节点开始不断向上回溯,找到的第一个不平衡的节点就不应该是Y这个节点。
所以依然是以Z为根的二叉树它是平衡的二叉树。相应的图3中以Z为根节点这棵二叉树保持平衡性, 相当于没有变化。

如果以Z为根节点保持平衡性的话, T1和T2它们的高度差不会超过1。假设T1和T2最大的高度值是H, 那么Z这个节点高度值就是H+1

在这里由于X也是保持平衡的。并且对于X来说它的平衡因子大于等0的, 也就是说左子树的高度大于等于右子树的高度。注意, 由于X也是保持平衡的, 所以X的平衡因子最大为1。也就是说X的平衡因子要么是0要么是1。对应就是T3这棵树的高度要么是H要么是H+1, 这样一来对于X这个节点来说它的高度值就是H+2。

我们在来看Y这个节点, 这个节点打破了平衡。它的左右子树高度差是大于1的。但是, 有个点需要注意的是这个高度差最大是2。这是为什么呢？

这是因为, 我们以Y节点为根的树添加了一个节点打破了平衡性。原来Y节点是平衡的, 现在我们添加一个节点之后, 如果不平衡了, 这个高度差只有可能为2。而不肯为3。这是因为我们只添加了一个节点, 不可能让左子树的高度一下就添加2, 所以在这种情况下由于Y这个节点不平衡了, 那么肯定是左子树比右子树大了2, 所以在这种情况下, T4的高度应该为H。


了解了这一点后, 我们在看一下旋转后的树, 对于T3这颗子树来说它的高度要么是H要么是H+1, 而T4它的高度是H。所以, 整体来看对于Y这个节点来说也是保持平衡的,
并且在这里Y这个节点, 它的高度值是H+2或者是H+1的, 具体是谁, 取决于T3的高度,
如果T3的高度是H+1, 那么Y节点的高度就是H+2, 如果T3的高度是H的话, T3和T4都是H, Y节点的高度值就是H+1。

不管Y这个节点高度是H+1还是H+2, 我们从X这个节点角度来看, X这个节点依然是平衡的。Y和Z两个节点高度差是不会超过1的。


```

[图2-3 [右旋转]]
![2-3](https://github.com/basebase/img_server/blob/master/common/bbt07.jpg?raw=true)
![2-3-1](https://github.com/basebase/img_server/blob/master/common/bbt08.png?raw=true)
