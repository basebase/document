### 栈

#### 栈的基本特点
  * 先进后出, 后进先出(LIFO, Last in First Out)
  * 除头尾节点外, 每个节点都有前驱和后继节点

#### 栈的基本操作
  * 压栈(push), 将数据放入栈顶。
  * 弹栈(pop), 将数据从栈顶移除。


通过下图来认识一下栈:

![avatar](https://github.com/basebase/img_server/blob/master/common/stack01.png?raw=true)



#### 栈的一些应用
举两个例子  
  1. 编辑器都有一个undo(即撤销)操作, 比如我们在IDEA中写代码写错了, 想要撤销到修改之前该怎么办呢? 就需要执行undo操作了。  
    **对于编辑器来说undo操作的原理是什么呢?**  
  其实就是依靠栈(Stack)来维护的, 如下过程:  
    * 输入**沉迷**, 编辑器就会记录这个动作。这个记录方式其实就是把这个动作放入栈中, 记录了**沉迷**

    * 然后输入**学习**, 编辑器的栈会在记录一次, 以同样的方式压入栈中。

    * 下面如果我要输入**无法**但却输入了**无天**, 栈中也记录了这个动作。发现输入出错后, 需要撤销这个操作, 这里需要怎么做?
      * 其实是从编辑器的这个栈中取出栈顶元素, 而栈顶记录的操作是输入出错的"无天", 执行撤销就是删除这两个字。也就是把这两个字从栈顶中移除。


  2. 在比如说我们系统程序的调用, 比如有A,B,C三个方法, A调用了B, B调用了C, 那么当A方法中执行到调用B方法的时候会将A方法压入系统栈, 当B调用C的时候会把B方法压入栈, 当C方法执行完毕后, 会从栈中找出上一次执行中断的方法再次执行, 直到栈中为空。如下图:  

  ![avatar](https://github.com/basebase/img_server/blob/master/common/stack02.png?raw=true)

  ![avatar](https://github.com/basebase/img_server/blob/master/common/stack03.jpg?raw=true)
