# 最长回文子串

## 题目

> 给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：

```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```


示例 2：

```
输入: "cbbd"
输出: "bb"
```

## 解法

### 暴力

**解法**

穷举所有子串，更新答案

**编码**

```java
public class Solution {
    public String longestPalindrome(String s) {
        int len = s.length();
        if (len < 2) {
            return s;
        }

        int maxLen = 1;
        int begin = 0;
        // s.charAt(i) 每次都会检查数组下标越界，因此先转换成字符数组
        char[] charArray = s.toCharArray();

        // 枚举所有长度大于 1 的子串 charArray[i..j]
        for (int i = 0; i < len - 1; i++) {
            for (int j = i + 1; j < len; j++) {
                if (j - i + 1 > maxLen && validPalindromic(charArray, i, j)) {
                    maxLen = j - i + 1;
                    begin = i;
                }
            }
        }
        return s.substring(begin, begin + maxLen);
    }

    /**
     * 验证子串 s[left..right] 是否为回文串
     */
    private boolean validPalindromic(char[] charArray, int left, int right) {
        while (left < right) {
            if (charArray[left] != charArray[right]) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
}
```

**性能分析**

- 时间：O(n^3)
- 空间：O(1)

### 动态规划

**解法**

分析回文串，若 b1 = "a1 s a2"，a1 == a2，则当s是回文时，b1是回文，s不是回文时，b1不是回文

dp[i]\[j] => 子串 i ~ j，是（1）否（0）是回文 

**编码**

```java
public String longestPalindrome(String s) {
    if (s == null || s.length() < 2) return s;
    // 动态规划， 
    // i : 0 ~ n
    // j : 0 ~ n
    char[] a = s.toCharArray();
    int len = s.length();
    boolean[][] dp = new boolean[len][len];

    for (int i = 0; i < len; i++) {
        dp[i][i] = true;
    }
    int maxLen = 1;
    int begin = 0;
    for (int j = 1; j < len; j++){
        for (int i = 0; i < j; i++) {
            if (a[i] != a[j]) {
                dp[i][j] = false;
            } else {
                if (j - i < 3) { // j - i + 1 < 4
                    dp[i][j] = true;
                } else if (dp[i + 1][j - 1]) {
                    dp[i][j] = dp[i + 1][j - 1];
                } 
                // 初始化时都是false,故不必修改
                // else {
                //     dp[i][j] = false;
                // }
            }
            if (maxLen < j - i + 1 && dp[i][j]) {
                begin = i;
                maxLen = j - i + 1;
            }
        }
    }
    return s.substring(begin, begin + maxLen);
}
```

**性能**

- 时间：O(n^2)
- 空间：O(n^2)

### 中心扩展法

**分析**

以每一个位置为中心，向两边扩展，同时更新结果

**编码**

```java
public String longestPalindrome(String s) {
    int len = s.length();
    if (s == null || len < 2) return s;

    int maxLen = 0;
    String res = s.substring(0, 1);
    for (int i = 0; i < len - 1; i++) {
        String oriStr = centerSpread(s, i, i);
        String opsStr = centerSpread(s, i, i + 1);

        String tempStr = oriStr.length() > opsStr.length() ? oriStr : opsStr;
        if (maxLen < tempStr.length()) {
            maxLen = tempStr.length();
            res = tempStr;
        } 
    }
    return res;
}
// 扩展 => {中心: (left + right) / 2 , 扩展：<---left, right---> }
public String centerSpread(String s, int left, int right) {
    int len = s.length();
    while (left >= 0 && right < len) {
        if (s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        } else {
            break;
        }
    }
    return s.substring(left + 1, right);
}
```

**性能**

- 时间：O(n^2)
- 空间：O(1)

### Manacher算法

**分析**

将之前的空隙替换为#（原字符串中不存在的字符用作结束），同时借鉴`中心扩展法`和`kmp思路`，利用数组将结果存储起来

最终结果不仅可以得出最长回文串，同时可以得到所有回文串

**编码**

```java
public String longestPalindrome(String s) {
    int len = s.length();
    if (len < 2) return s;

    String originStr = addBoundaries(s, '#');
    int oriLen = len * 2 + 1;
    int[] spreadArray = new int[oriLen];

    int maxLen = 0;
    int begin = 0;
    for (int i = 0; i < oriLen; i++) {
        spreadArray[i] = centerSpread(originStr, i);
        if (spreadArray[i] > maxLen) {
            maxLen = spreadArray[i];
            begin = (i - maxLen) / 2;
        }
    }
    return s.substring(begin, begin + maxLen); //-------------------
}

private int centerSpread(String s, int center) {
    int len = s.length();
    int i = center - 1;
    int j = center + 1;
    int step = 0;
    while (i >= 0 && j < len && s.charAt(i) == s.charAt(j)) {
        i--;
        j++;
        step++;
    }
    return step;
}
// 处理字符串，使用特殊字符填充原字符串空隙：abab => #a#b#a#b#
private String addBoundaries(String s, char divide) {
    int len = s.length();
    if (len == 0) {
        return "";
    }
    if (s.indexOf(divide) != -1) {
        throw new IllegalArgumentException("参数错误，您传递的分割字符，在输入字符串中存在！");
    }
    StringBuilder stringBuilder = new StringBuilder();
    for (int i = 0; i < len; i++) {
        stringBuilder.append(divide);
        stringBuilder.append(s.charAt(i));
    }
    stringBuilder.append(divide);
    return stringBuilder.toString();
}
```

**性能**

- 时间：O(n)
- 空间：O(2n)

