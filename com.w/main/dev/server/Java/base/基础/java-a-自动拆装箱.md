# 包装类型

Java中另外一种基本数据类型

- Java中还存在另外一种基本类型`void`，它也有对应的包装类 `java.lang.Void`

包装类型

- 出现的原因：

    - Java是一种面向对象语言，很多地方都需要使用对象而不是基本数据类型。

- | 基本数据类型 | 包装类    |
    | ------------ | --------- |
    | byte         | Byte      |
    | boolean      | Boolean   |
    | short        | Short     |
    | char         | Character |
    | int          | Integer   |
    | long         | Long      |
    | float        | Float     |
    | double       | Double    |

### 自动拆装箱原理

- 源代码

    ```java
    Integer integer = 12;
    int i = integer;
    ______________________
    Short s = 12;
    short ii = s;
    ______________________
    Long s = Long.valueOf(22L);
    long l = s.longValue();
    ______________________
    Byte b = 126;
    byte bb = b;
    ```

- 反编译

    ```java
    Integer integer = Integer.valueOf(12);
    int i = integer.intValue();
    ______________________
    Short s = Short.valueOf((short)12);
    short ii = s.shortValue();
    ______________________
    Long s = Long.valueOf(22L);
    long l = s.longValue();
    ______________________
    Byte b = Byte.valueOf((byte)12);
    byte bb = b.byteValue();
    ```

- 自动拆装箱实现方式基于

    > 包装类的**.valueOf()**，基本类型的**.xxxValue()**

### 自动拆装箱场景

- 基本数据类型放入集合类中
- 包装对象和基本类型比较的时刻
- 对Integer对象进行四则运算
- 三目运算符运算：第二，第三位操作数分别为基本类型和对象时，其中的对象就会拆箱为基本类型进行操作
- 参数与返回值不同时

### 自动拆箱与缓存

- 在Java中，==比较的是对象应用，而equals比较的是值。

    ```java
    public static void main(String[] args) {
        Integer num1, num2, num3, num4 ;
    
        num1 = -128;
        num2 = -128;
    
        num3 = 128;
        num4 = 128;
    
        System.out.println("when -128 <= integer <= 127");
        System.out.println(num1 == num2);
    
        System.out.println("when integer > 127 or integer < -128");
        System.out.println(num3 == num4);
    
    }
    //结果
    when -128 <= integer <= 127
    true
    when integer > 127 or integer < -128
    false
    ```

    

- 原因就和Integer中的缓存机制有关。在Java 5中，在Integer的操作上引入了一个新功能来节省内存和提高性能。整型对象通过使用相同的对象引用实现了缓存和重用。

    > 适用于整数值区间-128 至 +127。
    >
    > 只适用于自动装箱。使用构造函数创建对象不适用。

### 自动拆箱问题

包装对象的数值比较，不能简单的使用`==`

- 虽然-128到127之间的数字可以，但是这个范围之外还是需要使用`equals`比较。

空指针异常

- 前面提到，有些场景会进行自动拆装箱，同时也说过，由于自动拆箱，如果包装类对象为null，那么自动拆箱时就有可能抛出NPE。

大量拆箱浪费资源

- 如果一个for循环中有大量拆装箱操作，会浪费很多资源

#### Integer的缓存机制

- 创建对象之前先从IntegerCache.cache中寻找。如果没找到才使用new新建对象。

    ```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    System.out.println(Integer.valueOf(127) == Integer.valueOf(127));  // true
    ```

- 最大值127可以通过`-XX:AutoBoxCacheMax=size`修改

- 缓存通过一个for循环实现。从低到高并创建尽可能多的整数并存储在一个整数数组中。在Integer类第一次被使用的时候被初始化

```java
public static void main(String[] args) {
    Integer i1 = 127;
    Integer i2 = 127;

    Integer i3 = 129;
    Integer i4 = 129;

    Integer i5 = new Integer(127);
    Integer i6 = new Integer(127);

    System.out.println(i1 == i2); // true
    System.out.println(i3 == i4); // false

    System.out.println(Integer.valueOf(127) == Integer.valueOf(127)); // true

    System.out.println(i5 == i6); // false
}
```

```java
public static void main(String args[])
{
    Integer i1 = Integer.valueOf(127);
    Integer i2 = Integer.valueOf(127);
    Integer i3 = Integer.valueOf(129);
    Integer i4 = Integer.valueOf(129); // 自动装箱采用.valueOf(xx)实现，会在缓存中查找
    Integer i5 = new Integer(127); // 直接实例化，构建新对象，新建的永远都不是原有对象
    Integer i6 = new Integer(127);
    System.out.println(i1 == i2);
    System.out.println(i3 == i4);
    System.out.println(Integer.valueOf(127) == Integer.valueOf(127));
    System.out.println(i5 == i6);
}
```

#### 语言规范的缓存行为

- 如果一个变量p的值是：

    - -128至127之间的整数

    - true 和 false的布尔值

    - ‘\u0000’至 ‘\u007f’之间的字符
        - 将p包装成a和b两个对象时，可以直接使用a==b判断a和b的值是否相等

#### 其他整数类型的缓存机制

- 有ByteCache用于缓存Byte对象

    有ShortCache用于缓存Short对象

    有LongCache用于缓存Long对象

    有CharacterCache用于缓存Character对象

- 范围

    - `Byte`, `Short`, `Long`有固定范围: -128 到 127。
    - 对于`Character`, 范围是 0 到 127。
    - 除了`Integer`以外，这个范围都不能改变。