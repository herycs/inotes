# equals&==

- ==：
    - 基本数据类型，比较的是值
    - 引用数据类型比较的是地址（new的对象，==比较永远是false）
- equals：
    - 属于Object类的方法，如果我们没有重写过equals方法，那么它就是 ==
    - 字符串里面的equals被重写过了，比较的是值

Object中直接使用 == 比较，看一下Object类中的初始equals方法

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

String类中对equals的重写

```java
public boolean equals(Object anObject) {
    if (this == anObject) {// 地址相对那就一步到位，直接相等
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])// 存在不等情况，则返回false
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

