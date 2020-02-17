#### 选择排序(Selection Sort)

##### 选择排序基本思路

选择排序是一个O(N^2)的排序算法, 它和冒泡排序逻辑基本一致。除了交换次数不一样, 冒泡排序每次比较(最大或者最小元素)就需要立即交换, 而选择排序是把最小的元素找到后与之进行交换, 相对来说选择排序的交换次数较少, 一定程度上提高运算效率。

比如，你有下图中一组数据, 选择排序是如何排序的呢? 假设我们按照升序的方式排列

![1-1](https://github.com/basebase/img_server/blob/master/leetcode/array/array03.png?raw=true)



```text
1. 首先从第一个元素8去进行比较, 取出最小元素1, 就把索引4的元素和索引1的元素交换
更新后: 1, 6, 2, 3, 8, 5, 7, 4

2. 接着用第二个元素, 找到最小的元素索引位置2, 索引2和索引3的元素进行交换
更新后: 1, 2, 6, 3, 8, 5, 7, 4

3. 继续第三个元素, 进行比较找到最小元素索引位置3, 索引2和索引3的元素交换
更新后: 1, 2, 3, 6, 8, 5, 7, 4

4. 继续第四个元素, 进行比较找到最小元素索引位置7, 索引3的位置和索引7的位置进行交换
更新后: 1, 2, 3, 4, 8, 5, 7, 6

...中间过程类推, 最终, 我们的数组顺序必须为

out: 1, 2, 3, 4, 5, 6, 7, 8
```

以上就是选择排序的一个基本思路, 一直找到(最大或者最小元素索引位置)然后进行交换。
直到所有数组全部遍历完成。

##### 选择排序的具体实现

```java
public class SelectionSort<T extends Comparable> {

    public void selectionSort(T[] nums) {
        for (int i = 0; i < nums.length; i++) {
            int index = i;
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[i].compareTo(nums[j]) == 1)
                    index = j;
            }

            swap(nums, i, index);
        }
    }

    private void swap(T[] nums, int i, int j) {
        T tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }

    private String get(T[] nums) {
        StringBuilder res = new StringBuilder();
        for (int i = 0; i < nums.length; i++) {
            if (i == nums.length - 1)
                res.append(nums[i]);
            else
                res.append(nums[i]).append(", ");
        }

        return res.toString();
    }

    private void testMain() {
//        int[] nums = {10, 9, 8, 7, 6, 5, 4, 3, 2, 1};
        Float[] nums = {5.1f, 4.1f, 3.1f, 2.1f, 1.1f, 0.8f, 0.7f, 0.3f, 0.2f, 0f};
        System.out.println("排序前: " + get((T[]) nums));
        selectionSort((T[]) nums);
        System.out.println("排序后: " + get((T[]) nums));
    }

    public static void main(String[] args) {
        SelectionSort<Float> selectSort = new SelectionSort<>();
        selectSort.testMain();
    }
}
```
