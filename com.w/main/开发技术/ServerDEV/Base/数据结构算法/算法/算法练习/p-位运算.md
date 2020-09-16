# 位运算

#### [201. 数字范围按位与](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)

> 范围内的所有数字与的结果是两个数字的最长公共子前缀，例如下面的结果就是1001 0000
>
> n =  1 0 0  1 0 0 01
>
> m = 1 0 0  1 1 1 1

```java
class Solution {
    //思路：汉明距离，找到第一个小于m的低位为0的二进制数
    public int rangeBitwiseAnd(int m, int n) {
        if (m < 0 || n < 0) return 0;
        while (m < n){
            n &= (n - 1);//每次将最右边的1变为0
        }
        return n & m;
    }
}
```

