#### LeetCode数组问题

##### LeetCode 283 Move Zeros(移动零)

题目描述:  
给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。

示例:
```text
输入: [0,1,0,3,12]
输出: [1,3,12,0,0]
```
**说明**
1. 必须在原数组上操作，不能拷贝额外的数组
2. 尽量减少操作次数。


###### 解法1(暴力解法)
  1. 编译一次数据获取所有非0元素数据, 并按照顺序写入原先的数组中
  2. 更新剩下元素为0

[暴力解法]
![1-1](https://github.com/basebase/img_server/blob/master/leetcode/array/array01.png?raw=true)

```java

public void moveZeroes(int[] nums) {
    /***
     *  写法1: 暴力解法
     */

    int[] newNums = new int[nums.length];
    int index = 0;

    // 获取到所有非0的元素
    for (int i = 0 ; i < nums.length ; i ++)
        if (nums[i] != 0) {
            newNums[index] = nums[i];
            index ++;
        }


    // 重新对nums数组赋值
    for (int i = 0; i < nums.length; i ++) {
        if (i < index) {
            nums[i] = newNums[i];
        } else {
            nums[i] = 0;
        }
    }

}
```

###### 解法2(原地移动)
上面的暴力解法, 时间和空间的复杂度都是O(n)级别的。最明显的还是使用了一个O(n)的空间, 那么有什么办法可以不使用额外的辅助空间, 直接在原地完成对非0元素的移动呢?


1. 遍历数组, 遇到非0的元素直接放到前面的位置即可。非0元素是一定小于等于整个数组元素个数。所以完全不用担心这个赋值过程会覆盖掉任何有用的信息。

2. 如何记录位置呢? 我们使用一个变量k作为索引进行记录, 如果当前元素为0则k不更新, 如果元素不为0的话, 则将当前元素写入到索引k的位置, k进而更新下一个存放元素索引信息。

[原地修改解法]
![1-1](https://github.com/basebase/img_server/blob/master/leetcode/array/array02.jpg?raw=true)



```java
public void moveZeroes(int[] nums) {
    /***
     *  写法2: 原地移动
     */

    int k = 0;
    for (int i = 0; i < nums.length; i ++) {
        if (nums[i] != 0)
            nums[k++] = nums[i]; // 这里我直接使用nums[k++]来写入元素位置并维护k要写入的下一个索引位置, 如果看不明白, 可以分成两段来写
    }

    for (int i = k; i < nums.length; i ++)
        nums[i] = 0;
}
```

那么, 更新后的代码, 时间复杂度依旧是O(n)这是不可避免的, 但是空间复杂度变为O(1)了。没有使用任何的辅助空间。原地进行修改。


###### 解法3(原地移动)
经过上面的原地移动, 但是最后还需要从k的位置重新赋值为0。  
我们可以在把非0元素移动到前面的同时并且把为0的元素移至末尾, 减少最后的赋值操作呢?

```java
public void moveZeroes(int[] nums) {
    /***
     *  写法3: 原地移动
     */
    int k = 0;
    for (int i = 0; i < nums.length; i ++) {
        if (nums[i] != 0) {
            if (i != k) { // 如果整个数组都为0的话, 就不进行交换
                nums[k++] = nums[i];
                nums[i] = 0;
            } else {
                k ++;
            }
        }
    }
}
```
