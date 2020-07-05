# 算法 - 排序 - 交换类

# 1.冒泡排序

使用两个索引：i, j

[i]指向当前需要存放最小元素的位置

[j]指向当前与[i].value比较的元素

策略：i依次指向数组的每个元素，j负责找出{最小的元素，第2小的元素，，，第n小的元素}，随后交换 i, j 指向的值

**改进**

1. 当比较过程中出现没有交换的情况，则数据已经有序，后续的比较可以免了

    方法：设置标志位，当发生了交换则继续，未发生则跳出循环

    ```java
    boolean flag = true;
    for (int i = 0; i < nums.length && flag; i++) {
        flag = false;
        for (int j = 0; j < nums.length-1; j++) {
            if (nums[j] > nums[j]){
                SortUtils.swap(nums, j, j+1);
                flag = true;
            }
        }
    }
    ```

2. 数组中可能存在有序序列

    方法：增加遍历记录交换发生的最后一次的索引，这个索引记录的就是数组的乱序数组的最后边界

    ```java
    boolean flag = true;// flag = false,则数组已有序
    int arrBound = nums.length - 1;//无序数组边界，默认整个数组均乱序
    int lastSwapIndex = 0;//发生交换的最大边界值
    
    for (int i = 0; i < nums.length && flag; i++) {
        flag = false;
        for (int j = 0; j < arrBound; j++) {
            if (nums[j] > nums[j+1]){
                SortUtils.swap(nums, j, j+1);
                flag = true;
                lastSwapIndex = j;
            }
        }
        arrBound = lastSwapIndex;
    }
    ```

# 2.快速排序

