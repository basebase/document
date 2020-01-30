#### 并查集(union-find)


##### 什么是并查集?

并查集也是一种树结构, 它用于处理一些不交集的<strong>合并及查询问题。</strong>
以往的树结构都是父亲指向儿子, 而并查集是儿子指向父亲。

并查集支持下面两种操作:
  * 查找(find): 确定某个元素属于哪个子集。它可以被用来确定两个元素是否属于同一个子集。
  * 合并(Union): 将两个子集合并成一个子集。

<strong>也就是说，不支持集合的分离、删除。</strong>



先来说说<b>查找</b>究竟有什么用?   
举个例子: 几个家族进行宴会, 但是家族普遍长寿, 所以人数众多。由于长时间的分离以及年龄的增长, 这些人逐渐忘掉自己的亲人。只记得自己的爸爸是谁了。而最长者(称为"祖先")的父亲已经去世, 他只知道自己是祖先, 为了确定自己是哪个家族, 他们想出一个办法, 只要问自己的爸爸是不是祖先, 一层一层的往上问, 直到问到祖先。如果要判断两个人是否在同一个家族, 只要看两个人的祖先是不是同一个人就可以。

通过查找, 我们能清楚的知道两个人是否存在关系。 那么合并呢?

合并:  
宴会上, 一个家族的祖先突然对另外一个家族说: 我们两个家族交情这么好, 不如合成一家好了。另一个家族也欣然同意。


##### 并查集实现

##### 设计并查集接口
上面我们提到, 并查集只支持"查找"和"合并"两种操作, 所以我们接口中只设计该方法。

```java
public interface UF {

    // 并查集元素个数
    int getSize();

    boolean find(int p, int q);
    void union(int p, int q);
}
```


##### 并查集实现版本V1, Quick Find

现在我们要实现上面接口的实现, 一个是将两个元素合并在一起变成在一个集合中的元素(union), 另外一个就是检查两个元素是否是相连的(find)。

既然要判断是否所属同一个集合中或者合并元素进而同属于一个集合中, 所以我们可以在并查集内部数据做一个编号, 进而辨别。[如图1-1]

在这里0-9表示10个不同的数据, 当然这是一种抽象的表示, 具体可以想象这0-9这10个编号是10个人, 10部车或者10本书, 这都是更具具体业务来决定的。但是, 在并查集的内部我们只存储0-9这10个编号。它表示具体的10个元素。对于每一个元素并查集存储的是一个它所属于的集合ID。什么意思呢?

可以看到图[1-1]中, 元素[0, 2, 4, 6, 8]这几条数据所属的集合ID是0, 元素[1, 3, 5, 7, 9]所属的集合ID是1。

不同的ID值就是不同的集合所对应的编号。简单来说就是对这10条数据分成了2个集合。其中
[0, 2, 4, 6, 8]这5个元素在一个集合中, [1, 3, 5, 7, 9]这5个元素在另外一个集合中。


图[1-1]
![1-1](https://github.com/basebase/img_server/blob/master/common/unionFind01.png?raw=true)


从图[1-1]也能看出来, 其实就是利用数据来存储对应的id编号, 这种方式在查找中效率很高O(1), 但是在进行union的话就需要O(n)了。

比如说: 我现在要合并元素1和4, 可以看到元素1对应的集合id是1但是元素4对应的集合id是0, 在这种情况下, 将1和4这两个元素合并后, 1所属的集合和4所属的集合每一个元素相当于也连接了起来, 简单来说0和1的集合编号, 我们取其中一个进行覆盖让其都能关联在一起。

所以, 经过union之后, 就会变成图[1-2]这个样子

图[1-2]
![1-2](https://github.com/basebase/img_server/blob/master/common/unionfind02.png?raw=true)


```java
public class UnionFindV1 implements UF {

    private int[] id; // 集合编号

    public UnionFindV1(int size) {
        id = new int[size];

        /***
         * 在初始化的时候, 我们的元素都是独立的, 还没有某两个元素互相合并
         * 合并操作等我们构建好并查集之后进行union即可
         *
         * 现在我们初始化, 每个元素的编号都不一样
         *   第0个元素对应的集合编号是0
         *   第1个元素对应的集合编号是1
         *   ...依次类推
         *
         */
        for (int i = 0 ; i < id.length; i ++)
            id[i] = i;
    }

    @Override
    public int getSize() {
        return id.length;
    }

    @Override
    public boolean find(int p, int q) {

        /***
         * 首先查询p和q两个元素所属同一个集合编号
         *   O(1)查找
         */
        return find(p) == find(q);
    }

    // 查找元素p所对应的集合编号
    private int find(int p) {
        if (p < 0 || p >= id.length)
            throw new IllegalArgumentException("Error/");
        return id[p];
    }

    @Override
    public void union(int p, int q) {

        /***
         * 合并元素p和元素q所属的集合。
         *   需要循环所有元素进行替换O(n)
         */

        int pID = find(p);
        int qID = find(q);

        if (pID != qID) { // 这里只判断它们属于不同的集合中, 才进行合并。
            for (int i = 0; i < id.length; i++)
                if (id[i] == pID)
                    id[i] = qID;
        }
    }
}
```



##### 并查集实现版本V2, Quick Union

上个版本中, 我们实现了并查集的一种现实思路。我们实际使用数组进行模拟得到的结果叫做Quick Find, 也就是查找这个操作是非常快的。不过在标准的情况下, 并查集的实现思路是一种叫做Quick Union这样的实现思路。

Quick Union实现思路是如何的呢?   
具体就是将每一个元素, 看成是一个节点。而节点之间相连接形成一个树结构。不过这里的树结构和我们之前学习的树结构是不同的, 我们在并查集中实现的树机构是孩子指向父亲的。

什么意思呢? 如下图[1-3]:


图[1-3]
![1-3](https://github.com/basebase/img_server/blob/master/common/unionfind04.png?raw=true)

首先我们看"图例1", 我们有节点3和节点2, 如果要连接在一起, 指向方式是3指向2, 而2则是根节点, 由于根节点也有一个指针, 根节点指针只需要指向自己就可以。

这种情况下, 比如说对于"节点1"所代表的的元素要和节点3所代表的元素进行合并, 合并操作是怎么实现的呢? 实际上就是让1这个节点的指针指向3所在的这颗树的根节点, 也就是让节点1指向根节点2, 查看"图例2"。

当然了, 有可能在我们的并查集中存在一棵树, 如"图例3"中的[5, 6, 7], 其中6和7都是5的孩子, 现在如果我想让7这个节点和2这个节点进行合并, 实际上就是让7所在的根节点即5这个节点去指向2这个节点就可以了。当然了, 如果我想让7这个节点和3这个节点合并得到的结果也是一样的。实际上我们要做的是找到7这个节点根节点5指向3这个节点的根节点2。依然是根节点5指向根节点2。

这样的数据结构样子, 才是实际实现一个并查集的思路。  
在这种思路下, 我们具体的存储就发生了变化, 但其实还是非常简单的, 我们观察"图例3"中, 每一个节点其实只有一个指针, 也就是会指向另外一个元素, 关于指针的存储, 我们依然可以使用数组来实现。


对于这个数组, 我们可以把它称为parent, parent(i)表示的就是第i个节点指向那个节点, 所以之前虽然一直在说指针, 但实际存储的时候依然使用一个int型的数组就可以了。

这样一来, 在初始化的时候, 我们每一个节点都没有和其它的节点进行合并。所以每一个节点都指向了自己。


以10个元素为例子, 具体观察图[1-4], 每一个节点都是一个根节点。都指向自己。严格来说, 当前我们的并查集不是一棵树结构, 而是一个森林。"所谓的森林就是里面有很多的树。", 在初始的情况下, 现在我们的森林中就有10颗树。每颗树都只有一个节点而已。

具体我们重点来说说第2步, 第5步, 第9步。


第2步: union(4, 3)这个操作, 怎么做呢? 其实就是将4这个节点指向3这个节点就可以了。在我们的parent数组中反应出来就是parent(4)=3, 这就代表4这个节点它指向了3这个节点。

第5步: union(9, 4), 这个过程就是让9这个节点指向4这个节点所在的根节点。这里就有一个查询过程了, 我们就要看一下4这个节点指向了3这个节点, 3这个节点指向了8, 而8指向了自己说明8是根节点。我们就找到了4这个节点所在的根节点是8, 我们要做的就是让9这个节点指向8这个根节点就可以了。"在这里可以看出, 我们为什么不让9指向4这个节点呢? 因为这样指完以后就形成一个链表了, 那么我们的树整体的优势就体现不出来了。现在我们让9指向8, 如果我们要查询9这个节点对应的根节点是谁, 只需要进行一步查询。"

第9步: union(6, 2), 相应的我们要找到6这个节点的根节点是0, 我们在找到2这个节点的根节点是1, 相应的我们让6这个节点的根节点0指向2这个节点的根节点1就可以了。


通过这个模拟的过程, 我们的union的时间复杂度是一个O(h)级别的。其中这个h是树的深度大小。这个深度的大小在通常的情况下都比我们的元素个数n要小, 所以我们的union过程相对来说会快些。相对的代价就是查询则是树的深度大小。

图[1-4]
![1-4](https://github.com/basebase/img_server/blob/master/common/unionfind03.jpg?raw=true)


```java

public class UnionFindV2 implements UF {

    private int[] parent;


    public UnionFindV2(int size) {
        parent = new int[size];
        // 初始化, 相互之间没有共同集合, 大家都指向自己
        for (int i = 0 ; i < parent.length; i++)
            parent[i] = i;
    }

    @Override
    public int getSize() {
        return parent.length;
    }

    @Override
    public boolean find(int p, int q) {
        return find(p) == find(q);
    }

    /***
     * 查找过程, 查找元素p所对应的集合编号
     * O(h)复杂度, 其中h为树的高度。
     * @param p
     * @return
     */
    private int find(int p) {

        if (p < 0 || p >= parent.length)
            throw new IllegalArgumentException("Error");

        while (p != parent[p]) { // 当p和parent[p]相等也就是指向自己, 即根节点
            p = parent[p];
        }

        return p;
    }

    @Override
    public void union(int p, int q) {
        int pRoot = find(p);
        int qRoot = find(q);

        // 让p的根节点指向q的根节点
        if (pRoot != qRoot)
            parent[pRoot] = qRoot;
    }
}
```



##### 并查集实现版本V3, 基于size优化

在进行优化之前, 我们先来对之前两个版本union-find进行一个简单测试。


```java

public class UnionFindTest {

    private static double testUF(UF uf, int m) {

        int size = uf.getSize();
        Random random = new Random();

        long startTime = System.nanoTime();

        // m次合并性能测试
        for (int i = 0; i < m; i ++) {
            int a = random.nextInt(size);
            int b = random.nextInt(size);
            uf.union(a, b);
        }

        // m次查询性能测试
        for (int i = 0; i < m; i ++) {
            int a = random.nextInt(size);
            int b = random.nextInt(size);
            uf.find(a, b);
        }


        long endTime = System.nanoTime();

        return (endTime - startTime) / 1000000000.0;
    }

    public static void main(String[] args) {
        int size = 10000;
        int m = 10000;

        UnionFindV1 unionFindV1 = new UnionFindV1(size);
        System.out.println("UnionFindV1: " + testUF(unionFindV1, m) + " s");

        UnionFindV2 unionFindV2 = new UnionFindV2(size);
        System.out.println("UnionFindV2: " + testUF(unionFindV2, m) + " s");

    }
}
```

```text

当size=10000, m=10000时候, 我的输出如下:
UnionFindV1: 0.11981561 s
UnionFindV2: 0.068716989 s

差距并不是很大。由于UnionFindV1合并操作是O(n)级别的, 这个n就是size的值。为了显示它们之间的差距更加明显。尝试修改size的值。

当size = 100000, m=10000时候, 我的输出如下:
UnionFindV1: 0.502680044 s
UnionFindV2: 0.005111976 s

这次的运行, 二者之间的差距就显示的非常大了。

但是V2真的就比V1好吗?
当size=100000, m=100000, 我的输出如下:
UnionFindV1: 7.837964178 s
UnionFindV2: 15.23050521 s

可以看到UnionFindV2比UnionFindV1还要慢了。
由于UnionFindV1整体就是使用一个数组, 我们的合并就是对这个一个连续空间进行循环操作JVM有比较好的优化所以运行速度会比较快, 相应的UnionFindV2查询的过程是一个不断索引的过程, 它不是一个顺序的访问一片连续空间过程。要在不同地址之间进行跳转因此速度会慢一些。第二个原因就是UnionFindV2的find过程是O(h)比我们UnionFindV1要高。

```

好的, 当我们发现V2的版本小于V1的时候, 我们去观察一下union过程中, 更多的元素被组织在一个集合中, 所以我们得到的树是非常大的, 相应的深度也会非常的高。这就使得在后续进行m次find操作它的时间性能消耗也会非常的高。


我们在进行union操作的时候, 就直接将p元素的根节点指向q元素的根节点。
我们没有充分考虑p和q这两个元素, 所在的树的特点是如何的。

那么在优化UnionFindV2之前, 我们先来看看下面这张图。


![1-5](https://github.com/basebase/img_server/blob/master/common/unionfind05.jpg?raw=true)

通过上图, 我们发现, 现在的并查集在实现union过程中并没有去判断两个元素的树结构, 很多时候这个合并过程会不断增加树的高度。甚至在某些极端的情况下我们得到的是一个链表形状。具体如何解决呢? 一个简单解决方案就是考虑"size", 当前这棵树有多少个节点。简单来说就是"让节点个数小的树它的根节点去指向节点个数多的那棵树的根节点。"这样处理之后, 高概率它的深度会比较低。



```java

public class UnionFindV3 implements UF {

    private int[] parent;
    private int[] sz; // 记录树的节点个数, sz[i]表示以i为根节点的集合中元素个数


    public UnionFindV3(int size) {
        parent = new int[size];
        sz = new int[size];

        // 初始化, 相互之间没有共同集合, 大家都指向自己
        for (int i = 0 ; i < parent.length; i++) {
            parent[i] = i;
            sz[i] = 1; // 初始化, 每棵树的高度都为1
        }

    }

    @Override
    public int getSize() {
        return parent.length;
    }

    @Override
    public boolean find(int p, int q) {
        return find(p) == find(q);
    }

    /***
     * 查找过程, 查找元素p所对应的集合编号
     * O(h)复杂度, 其中h为树的高度。
     * @param p
     * @return
     */
    private int find(int p) {

        if (p < 0 || p >= parent.length)
            throw new IllegalArgumentException("Error");

        while (p != parent[p]) { // 当p和parent[p]相等也就是指向自己, 即根节点
            p = parent[p];
        }

        return p;
    }

    @Override
    public void union(int p, int q) {
        int pRoot = find(p);
        int qRoot = find(q);



        // 让p的根节点指向q的根节点
        if (pRoot != qRoot) {

            // 判断两个树的元素个数, 让元素个数小的根节点指向元素个数多的根节点。
            // 根据两个元素所在树的元素个数不同判断合并方向
            // 将元素个数少的集合合并到元素个数多的集合上

            if (sz[pRoot] < sz[qRoot]) {
                parent[pRoot] = qRoot;
                sz[qRoot] += sz[pRoot]; // 维护sz数组的值, 我们让qRoot值加上pRoot的值, 因为它们已经合并了。
            } else {
                parent[qRoot] = pRoot;
                sz[pRoot] += sz[qRoot];
            }

        }



    }
}
```

现在, 我们把UnionFindV3加入进行测试, 我这边输出如下时间:

```text
UnionFindV1: 7.297955247 s
UnionFindV2: 14.029683212 s
UnionFindV3: 0.030103227 s

可以发现, 我们UnionFindV3加入size的优化后, 效率就提升很多了。
加入size后, 我们并查集保证树的深度是非常浅的。
```
