#### 插入排序(Insertion Sort)

##### 插入排序基本思路


假设我们要对下图一组数据进行排序。插入排序是如何实现的呢？
![1-1](https://github.com/basebase/img_server/blob/master/leetcode/array/array03.png?raw=true)

需要注意的是, 插入排序是从索引1也就是第2个元素开始, 如果只有一个元素默认即为有序的。如果发现当前元素大于上一个元素可以结束此次循序。也就是说可以提前结束循环。

```text

我们这里按照从小到大进行排序。

1. 从数组索引1开始进行比较, 获取到索引0进行判断, 发现6比8小进行交换。
更新后: 6, 8, 2, 3, 1, 5, 7, 4

2. 从数组索引2开始比较, 获取索引2元素值为2, 和索引1的元素8比较发现2比8小进行交换, 但是并没有结束, 继续往前推判断, 和索引0的元素进行比较判断, 发现比元素6更小, 进行交换
更新后: 2, 6, 8, 3, 1, 5, 7, 4

....以此类推

最终输出: 1, 2, 3, 4, 5, 6, 7, 8

```

##### 插入排序实现

```java


public void insertionSort(int[] nums) {
    for (int i = 1; i < nums.length ; i ++) {

        /**
         *  这里一定是要大于0的, 如果>=0的话, 索引到位置0的时候然后减1个元素, 数组就会抛出异常...
         *  java.lang.ArrayIndexOutOfBoundsException: -1
         */
        for (int j = i ; j > 0 && nums[j] < nums[j - 1] ; j --) {
            swap(nums, j - 1, j);
            /***
             *  这里的判断看起来比较冗余, 我们可以直接在for循环条件加上,
             *  如果当前元素小于上一个元素才进行循环
             */
//                if (nums[j] < nums[j - 1]) {
//                    swap(nums, j - 1, j);
//                } else if (nums[j] > nums[j - 1]) {
//                    break;
//                }
        }
    }
}
```
