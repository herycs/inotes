

# 全排列

> print() 打印数组元素
>
> swap() 交换 i，j 索引位置的值

递归-核心代码

```java
public static void permutation(int[] nums, int m) {
    if (m == nums.length) {
        print(nums);
    } else {
        for (int i = m; i < nums.length; i++) {
            swap(nums, i, m);
            permutation(nums, m + 1);
            swap(nums, i, m);
        }
    }
}
```

完整代码

```java
public class Main{
    public static void main(String[] args) {
        int[] nums = new int[]{1, 2, 3};
        permutation(nums, 0);
    }
    // 全排列
    public static void permutation(int[] nums, int m) {
        if (m == nums.length) {
            print(nums);
        } else {
            for (int i = m; i < nums.length; i++) {
                swap(nums, i, m);
                permutation(nums, m + 1);
                swap(nums, i, m);
            }
        }
    }
    // 交换
    public static void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    // 打印
    public static void print(int[] nums) {
        for (int i = 0; i < nums.length; i++) {
            System.out.print(nums[i]);
        }
        System.out.println();
    }
}
```



