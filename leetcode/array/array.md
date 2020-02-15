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


##### LeetCode 27 Remove Element(移除元素)
题目描述:  
给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

示例1:

```text
给定 nums = [3,2,2,3], val = 3,
函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。
你不需要考虑数组中超出新长度后面的元素。
```

**分析:**
  1. 这个删除并非说减少数组长度, 而是逻辑上的删除
  2. 非删除元素顺序, 可以不按照原有的顺序
  3. 对被删除后的数据不用移动要数组末尾或者保留保留原来的数值


###### 解法1(原地删除)
```java
public int removeElement(int[] nums, int val) {
    int k = 0;
    for (int i = 0 ; i < nums.length; i ++) {
        if (nums[i] != val)
            nums[k++] = nums[i];
    }

    return k;
}
```

基于上面的原地移动的一个思想, 我们可以利用k来获取要写入的元素索引位置。  
时间复杂度: O(n) , 空间复杂: O(1)



###### 解法2(原地删除)
解法2只是对解法1的一个补充, 思考一下, 如果要删除的元素很少, 使用解法1的代码就会
产生很多没必要的移动复制。

例如:
```text
num = [1，2，3，5，4], val=4
遇到这样的数组, 我们会对前面4个元素做不必要的复制操作。

再比如: num=[4，1，2，3，5], val = 4。似乎没必要将[1, 2, 3, 5]向左移一位。
因为问题已经说明元素顺序可以改变
```

那么, 如何解决这样的问题呢?

当我们nums[i] == val时, 我们可以将当前元素与最后一个元素进行交换, 并释放最后一个元素。这样就使数组大小减小了1。

```java
public int removeElement(int[] nums, int val) {
    /***
     * 写法2
     */
    int i = 0;
    int n = nums.length ;

    while (i < n) {
        if (nums[i] == val)
            nums[i] = nums[--n];
        else
            i++;
    }
    return n;
}
```

注意: 如果最后一个被交换的元素也是要移除的值, 不用担心。下一次迭代依然会检查这个元素。【因为我们的i元素并没有累加, 所以还是当前元素, 这样又会从后面更新元素上来】。

时间复杂度O(n) , 空间复杂的O(1)
虽然整体复杂度没有变化, 但是赋值操作的次数等于要删除的元素的数量。因此，如果要移除的元素很少，效率会更高。




##### LeetCode 26 Remove Duplicated from Sorted Array(删除排序数组中的重复项)


题目描述:  
给定一个排序数组，你需要在原地删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

示例:
```text
给定数组 nums = [1,1,2],
函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。
你不需要考虑数组中超出新长度后面的元素。
```

**分析:**
1. 删除重复的数据
2. 删除后的元素顺序必须也保证有序


###### 解法1(原地删除)

使用变量k来记住要写入的位置。

```java
public int removeDuplicates(int[] nums) {

    if (nums == null)
        return 0;
    if (nums.length <= 1)
        return nums.length == 0 ? 0 : 1;

    int k = 1;
    /***
     *  如果下一个元素依旧和上一个元素相同, 这里我们的k变量是不会更新的, 等到下一个不同的元素我们会更新k的索引位置信息
     */
    for (int i = 1; i < nums.length; i ++) {
        if (nums[i] != nums[k - 1])
            nums[k++] = nums[i];
    }

    return k;
}
```
