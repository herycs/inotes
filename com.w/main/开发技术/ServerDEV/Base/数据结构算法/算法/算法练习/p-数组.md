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

