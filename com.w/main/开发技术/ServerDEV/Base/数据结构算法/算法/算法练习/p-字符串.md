# 字符串

## 回文

以尾字符分类

以首尾字符分类

以首字符分类

####  [剑指 Offer 45.最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

Hash存储出现过的字符索引，用于更新左边界

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
      if (s == null || s.length() == 0) return 0;
        int res = 0;
        HashMap<Character, Integer> map = new HashMap<>();
        for (int i = 0, j = -1; i < s.length(); i++) {//其中j为左边界，i为右边界
            if (map.containsKey(s.charAt(i)))
                j = Math.max(j, map.get(s.charAt(i)));//更新左边界，取最大值是因为，重复字符的位置可能在当前序列的中间
            map.put(s.charAt(i), i);//更新新的索引
            res = Math.max(res, (i - j));
        }
        return res;  
    }
}
```

#### [剑指 Offer 45.把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

```java
class Solution {
    public String minNumber(int[] nums) {
        if (nums == null || nums.length == 0) return "";
        String[] str = new String[nums.length];
        for (int i = 0; i < str.length; i++) {
            str[i] = String.valueOf(nums[i]);
        }
        Arrays.sort(str, (a, b)->((a + b).compareTo(b + a))); // 重新大小规则
        StringBuilder builder = new StringBuilder();
        for (String s : str){
            builder.append(s);
        }
        return builder.toString();
    }
}
```

#### [1177. 构建回文串检测](https://leetcode-cn.com/problems/can-make-palindrome-from-substring/)

```java
class Solution {
    public List<Boolean> canMakePaliQueries(String s, int[][] queries) {
        List<Boolean> list = new ArrayList<>();
        if (s == null || s == "" || queries == null || queries.length < 1 || queries[0].length < 2) return list;
        
        int[] state = new int[s.length()];
        state[0] = 1 << (s.charAt(0) - 'a');
        for (int i = 1; i < s.length(); i++){
            state[i] = state[i - 1] ^ (1 << (s.charAt(i) - 'a'));
        }
        for (int i = 0; i < queries.length; i++){
            int left = queries[i][0];
            int right = queries[i][1];
            int k = queries[i][2];
            
            list.add(checkString(state, left, right, k));
        }
        return list;
    }
    
    public boolean checkString(int[] state, int left, int right, int k){
        int singleCharCount = 0;
        int temp = (left == 0 ? state[right] : (state[left - 1] ^ state[right]));  
        while (temp > 0){
            singleCharCount += temp & 1;
            temp >>= 1;
        }   
        return (singleCharCount >> 1) <= k;
    }
}
```

