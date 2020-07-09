# API

# String

## charAt

用于给出位置上的字符 

## substring

 substring（int start,int end）方法用于返回一个字符串的子字符串

```java
public class StringDemo03 {
    public static void main(String[] args) {
        //            01234567890123
        String str = "www.oracle.com";
        /*
         * 使用数字表示范围，都是含头不含尾，
         * 例如从4开始，到10结束
         */
         String sub = str.substring(4,10);
         System.out.println(sub);//oracle
         /*
          * 只有一个数字表示从这开始一直到结束
          */
         String sub1 = str.substring(4);
         System.out.println(sub1);//oracle.com
    }
}
```

## indexOf

 (查找给定字符串在当前字符串的位置，返回第一个字母所在下标)

```java
public class StringDemo {
    public static void main(String[] args){
        String str = "thinking in Java";

        int index = str.indexOf("in");
        System.out.println("index="+index);//index=2

        int index1 = str.indexOf("in",3);//从第4个字符开始寻找in
        System.out.println("index1="+index1); //index1=5

        int index2 = str.lastIndexOf("in");//从后往前查找
        System.out.println("index2="+index2);//index2=9
    }
}
```

## trim

去除字符串周围空白

## String [] split(regex)

## startsWith

是否以指定字符开头

## endsWith

是否以指定字符结尾

## boolean contains(str)

字符串中是否包含某一个字符串

## int compareTo(String )

对两个字符串进行自然顺序的比较

## 大小写转换

String toUpperCase() 将指定字符串全部转换为大写

String toLowerCase() 将指定字符串全部转换为小写

## valueOf

用于将其他类型转换为字符串类型

##  toCharArray()

转换为字符数组

## String(byte [])

 将字节数组转换成字符串，*String(byte [],offset,count) 将字节数组中的一部分转换成字符串*

# 集合

## Stack

### .size()

栈当前深度

## Deque

## Map

## Set

## List