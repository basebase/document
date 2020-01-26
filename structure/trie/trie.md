#### 字典树(Trie)


##### 什么是字典树?



Trie树, 又叫字典树、前缀树(Prefix Tree)、单词查找树或键树, 是一种多叉树结构。
Trie通常只用来处理字符串。

<strong>Trie特性:</strong>  
  1. 根节点不包含字符, 除根节点外每一个子节点都包含一个字符。
  2. 从根节点到某一个节点, 路径上经过的字符连接起来, 为该节点对应的字符串(单词)。
  3. 每个节点的所有子节点包含的字符互不相同。
  4. 通常在实现Trie的时候, 会在节点结构中设置一个标识, 用来标识该节点处是否构成一个单词

可以看出, Trie树的关键字一般都是字符串, 而且Trie树把每个关键字保存在一条路径上, 而不是一个节点中, 另外, 两个有公共前缀的关键字, 在Trie树中前缀部分的路径相同, 所以Trie树又叫做前缀树(Prefix Tree)。


<strong>Trie优缺点:</strong>  
  Trie树的核心思想是空间换时间, 利用字符串的公共前缀来减少无谓的字符串比较以达到提高查询效率的目的。

  <strong>优点:  </strong>  
  1. 插入和查询的效率很高，都为O(w)，其中w是待插入/查询的字符串的长度
  2. Trie树中不同的关键字不会产生冲突。
  3. Trie树只有在允许一个关键字关联多个值的情况下才有类似hash碰撞发生。
  4. Trie树不用求 hash 值，对短字符串有更快的速度。通常，求hash值也是需要遍历字符串的。
  5. Trie树可以对关键字按字典序排序。

  <strong>缺点:  </strong>
  + 当 hash 函数很好时，Trie树的查找效率会低于哈希搜索。
  + 空间消耗比较大。

目前对hash不了解没关系, 只要有个大概的印象就可以了。

假如现在有100万条数据, 如果使用Trie查找, 就和有多少条目没有关系。

|    数据结构    | 时间复杂度 |      备注 |
| -----------  |       --- |      --- |
| Trie         |      O(w) |    其中w为字符串长度
| BST          |   O(logn) |    


##### Tire数据结构

在考察一个字符串或者单词看成是一个整体, 但是Tire却打破了这种方式, 它以一个字母为单位拆分存储, 从根节点开始一直到叶子节点去遍历, 遍历到一个叶子节点就形成一个单词。

如图[1-1]中, 可以看到存储了4个单词, 分别是{"cat", "dog", "deer", "panda"}

图[1-1]
![1-1](https://github.com/basebase/img_server/blob/master/common/trie.png?raw=true)

我们要查询任何一个单词, 从根节点出发只需要经过这个单词有多少个字母, 过了多少个节点, 最终达到叶子节点。就成功查找到单词。这样的数据结构就叫做Trie。


###### Trie每一个节点是如何定义的？

由于我们的英文字母有26个, 所以每一个节点都有26个指向下一个节点的指针, 只不过我们图[1-1]中没有画那么多而已。

所以在Trie中节点大概定义如下
```java
class Node {
  char c ; // 每个节点装载一个字母
  Node[26] next ; // 装载26个指针
}
```

不过不同的场景下, 26个指针可能是富裕的, 有可能是不够的。
比如说, 每个节点下面跟26个孩子, 但是并没有考虑大小写的问题。如果我们设计的Trie要考虑大写的话, 相应的有52个指针。但是, 如果我们的Trie设计的更加复杂, 比如说装载了网址或者邮件地址, 相应的有一些字符也应该计算在内, 如: "@,:,\_-"等等。

所以通常并不会固定指针数量, 除非该场景固定就26个字母。  
所以我们需要<strong>每一个节点都有若干个指向下一个节点的指针。</strong>

```java

class Node {
  char c ;
  Map<char, Node> next ;
}
```

其实, 我们从根节点找到下一个节点的过程中, 我们就已经知道这个字母是谁了, 换句话说, 我从根节点来搜索"cat"这个词, 之所以能够来到这个节点, 是因为在根节点就知道我的下一个节点要到'c'所在的这个节点中。

所以, 在我们的设计中, 可以不存储这个字符

```java
class Node {
  Map<char, Node> next ;
}
```

不过上述的设计还是有问题, Trie从根节点一直到叶子节点才到了一个单词的地方。
比如我们查询到了't'我们就找到了"cat"这个词, 我们查询到了'g'我们就找到了"dog"这个词, 以此类推, <strong>不过在英语中有些单词可能是另外一个单词的前缀</strong>

比如说: "pan"这个单词, 如果我们这个Trie中既要存储"pan"又要存储"panda"那么怎么办呢? 此时这个"pan"它的结尾'n'并不是叶子节点, 正因为如此, 每一个节点都需要一个标识,这个标识来告诉大家当前这个节点是否是某一个单词的结尾, 某一个单词的结尾光靠叶子节点是无法区分出来的, 所以我们设计应该在加入一个字段代表是否为一个单词的结尾。

```java
class Node {
  boolean isWord ;
  Map<char, Node> next ;
}
```


##### 实现Trie

###### 构建Trie

```java

public class Trie {
/**
 * 更具上面所述, 构建我们的Node
 */
  private class Node {
    public boolean isWord;
    public Map<Character, Node> next;

    public Node(boolean isWord) {
        this.isWord = isWord;
        this.next = new TreeMap<>();
    }

    public Node() {
        this(false);
    }
  }

  // 根节点
  private Node root ;
  private int size ;

  // 初始化节点信息
  public Trie() {
    this.root = new Node();
    this.size = 0;
  }

  public int getSize() {
    return size;
  }
}
```


###### 向Trie添加元素

```java

/**
 * 向Trie中添加一个新的单词word
 * @param word
 */
public void add(String word) {
    Node cur = root;
    for (int i = 0 ; i < word.length(); i ++) {
        char c = word.charAt(i);
        if (cur.next.get(c) == null) // 如果下一个节点不存在字符就添加, 如果存在不做任何操作
            cur.next.put(c, new Node());

        cur = cur.next.get(c); // 重新赋值, 这样就到叶子节点但是有可能是某个非叶子节点
    }

    // 结束之后, 不能直接就size++, 需要判断是否之前就添加过该单词了, 就判断尾巴是否为true
    if (!cur.isWord) {
        cur.isWord = true;
        size ++;
    }
}

// 添加元素递归版
public void addRE(String word) {
    addRE(word, 0, root);
}

// 添加元素递归版
private void addRE(String word, int index, Node node) {

    if (index == word.length()) {
        if (!node.isWord) {
            node.isWord = true;
            size ++;
        }

        return ;
    }

    char c = word.charAt(index);
    if (node.next.get(c) == null)
        node.next.put(c, new Node());
    addRE(word, ++index, node.next.get(c));
}
```


###### 查询单词是否在Trie中

```java
/**
 * 查询单词是否在trie中
 * @param word
 * @return
 */
public boolean contains(String word) {
    Node cur = root;
    for (int i = 0 ; i < word.length(); i ++) {
        char c = word.charAt(i);
        if (cur.next.get(c) == null) // 如果不存在查找的单词字母, 则直接返回
            return false;

        cur = cur.next.get(c);
    }

    // 记住, 这里计算遍历出来后也不能直接返回true, 比如一开说的pan是panda前缀, 如果我们没有添加pan却返回了true就有问题了
    //        return true;
    return cur.isWord; // 正确的方式直接返回当前节点的标识
}

// 查询单词是否在trie中, 递归写法
public boolean containsRE(String word) {
    return containsRE(word, 0, root);
}

private boolean containsRE(String word, int index, Node node) {
    if (index == word.length()) {
        return node.isWord;
    }

    char c = word.charAt(index);
    return node.next.get(c) == null ? false : containsRE(word, ++index, node.next.get(c));
}
```


###### 前缀查询

几乎和查询逻辑是一样的, 只不过我们不需要按照isWord返回, 如果我们能顺利退出循环, 就表示我们能查询到该字符串的前缀。

```java

// 查询Trie中有单词以prefix为前缀
public boolean isPrefix(String prefix) {
    Node cur = root;
    for (int i = 0; i < prefix.length(); i ++) {
        char c = prefix.charAt(i);
        if (cur.next.get(c) == null)
            return false;

        cur = cur.next.get(c);
    }

    return true;
}

// 查询Trie中有单词以prefix为前缀(递归写法)
public boolean isPrefixRE(String prefix) {
    return isPrefixRE(prefix, 0, root);
}

private boolean isPrefixRE(String prefix, int index, Node node) {
    if (index == prefix.length()) {
        return true;
    }
    char c = prefix.charAt(index);
    return node.next.get(c) == null ? false : isPrefixRE(prefix, ++index, node.next.get(c));
}
```
