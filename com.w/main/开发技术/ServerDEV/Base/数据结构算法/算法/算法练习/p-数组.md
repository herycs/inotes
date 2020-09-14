# 数组

#### [718. 最长重复子数组](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

```java
class Solution {
    public int findLength(int[] A, int[] B) {
        if (A == null || A.length == 0 || B == null || B.length == 0) return 0;
        
        int lenA = A.length;
        int lenB = B.length;
        int[][] dp = new int[lenA + 1][lenB + 1];
        int res = 0;
        for (int i = lenA - 1; i >= 0; i--){
            for (int j = lenB - 1; j >= 0; j--){
                dp[i][j] = A[i] == B[j] ? dp[i + 1][j + 1] + 1 : 0;
                res = Math.max(res, dp[i][j]);
            }
        }
        return res;
    }
}
```

#### 数组覆盖

> 长度为L的绳子和一个数组
>
> 求绳子最多覆盖其中几个点

1. 定长双指针（right为当前位置，left = [right - L] 指向当前能覆盖的最开始的位置）
2. 快慢双指针（left，right不断后移，维持left 和 right的间距小于 L ）
    1. 当right - left <= L，更新值 right++
        1. right - left > L 保持原有值不更新，这时left 和 right之间覆盖的就是当前情况下的解
        2. left++，循环

```java
public static int maxPoint(int[] arr, int L) {
    if (arr == null || arr.length == 0 || L <= 0) return -1;

    int left = 0;
    int right = 0;
    int res = 0;
    while (left < arr.length){
        while (right < arr.length && (arr[right] - L) <= arr[left]){
            right++;
        }
        res = Math.max(res, right - left);
    }
    return res;
}
```

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        HashMap<Integer, Integer> preSumSeq = new HashMap<>();
        preSumSeq.put(0, 1);
        int count = 0;
        int preSum = 0;
        for (int i = 0; i < nums.length; i++) {
            preSum += nums[i];
            if (preSumSeq.containsKey(preSum - k)) {
                count += preSumSeq.get(preSum - k);
            }
            preSumSeq.put(preSum, preSumSeq.getOrDefault(preSum, 0) + 1);
        }
        return count;
    }
}
```

