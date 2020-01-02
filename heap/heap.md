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
  1. 结构性质  
    + 二叉堆是一棵完全二叉树
    + 


![avatar](https://github.com/basebase/img_server/blob/master/common/heap01.png?raw=true)
