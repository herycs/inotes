# 括号匹配问题

## 题目

> 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。
>
> 有效字符串需满足：
>
> 左括号必须用相同类型的右括号闭合。
> 左括号必须以正确的顺序闭合。
> 注意空字符串可被认为是有效字符串。

分析：

栈 + HashMap(维护括号配对关系)

## 编码

```java
class Solution {
  public boolean isValid(String s) {
    if (s.length() < 2) return false;// single char is not legally params
    Stack<Character> stack = new Stack<Character>();
    HashMap<Character, Character> originCharMap = new HashMap<Character, Character>();
    originCharMap.put('}', '{');
    originCharMap.put(')', '(');
    originCharMap.put(']', '[');

    char tempChar = ' ';
    char topCharOfStack = ' ';
    for (int i = 0; i < s.length(); i++) {
        tempChar = s.charAt(i);
        if (originCharMap.containsKey(tempChar)) {
            topCharOfStack = stack.isEmpty() ? '#' : stack.pop();
            if (topCharOfStack != originCharMap.get(tempChar)) 
                return false;
        } else {
            stack.push(tempChar);
        }
    }
    return stack.isEmpty();
  }
}
```

## 分析

复杂度：

- 时间：O(n)
- 空间：O(n)