# 算法 - 排序 - 插入

# 1.直接插入排序

- 已排序序列中从后向前扫描，找到相应的位置并插入
- 举例：摸牌

```java
private static int[] InsertDirect(int[] data) throws Exception {
        if (data == null)
            throw new Exception("待排序列为空");
        int[] nums  = new int[data.length+1];
        for (int i = 0; i < data.length; i++) {
            nums[i+1] = data[i];
        }

        int i, j;
        for (i = 2; i < nums.length; i++) {
            nums[0] = nums[i];//保存待查数据
            for (j = i-1; nums[0] < nums[j]; j--) {
                nums[j + 1] = nums[j];//记录后移
            }
                nums[j+1] = nums[0];//插入正确位置
        }
        return nums;
    }
```



# 2.折半插入排序

```java
private static void binarySort(int[] nums) {
        int low, height, mid;
        for (int i = 2;  i< nums.length; i++) {
            if (nums[i] < nums[i-1]){
                nums[0] = nums[i];
                low = 1;
                height = i-1;
                while (low <= height){
                    mid = (low+height) / 2;
                    if (nums[0] < nums[mid]){
                        height = mid-1;
                    }else {
                        low = mid+1;
                    }
                }
                for (int j = i-1; j >= low; j--) {
                    nums[j + 1] = nums[j];//后移元素
                }
                nums[low] = nums[0];//插入到正确位置
            }
        }
    }
```



# 3.希尔排序