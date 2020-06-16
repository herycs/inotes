# 数据类型

# 基本数据类型

### 8种基本数据类型

- char
- boolean
- byte, short, int, long
- float, double

#### 整型中byte、short、int、long的取值范围

-　byte：byte用1个字节来存储，范围为-128(-2^7)到127(2^7-1)，
    -　默认值为0。
-　short：short用2个字节存储，范围为-32,768 (-2^15)到32,767 (2^15-1)
    -　默认值为0，一般情况下，因为Java本身转型的原因，可以直接写为0。
-　int：int用4个字节存储，范围为-2,147,483,648 (-2^31)到2,147,483,647 (2^31-1)
    -　默认值为0。
-　long：long用8个字节存储，范围为-9,223,372,036,854,775,808 (-2^63)到9,223,372,036, 854,775,807 (2^63-1)
    -　long类型的默认值为0L或0l，也可直接写为0。

#### char取值范围

> 16位Unicoode存储

- 最小值是 **\u0000**（即为0）；
- 最大值是 **\uffff**（即为65,535）；

#### 什么是浮点型？

- 1的二进制是01，1.0/2=0.5，那么，0.5的二进制表示应该为(0.1)，以此类推，0.25的二进制表示为0.01

#### 什么是单精度和双精度？

- 单精度浮点数在计算机存储器中占用4个字节（32 bits）
- 双精度浮点数(double)使用 8字节（64 bits） 来存储一个浮点数

#### 为什么不能用浮点型表示金额？

- 计算机中保存的小数其实是十进制的小数的近似值，并不是准确值

# String

## 为何是final

> final修饰的**引用**被限定为**不可变**

1. 实现字符串常量池
2. 安全，使用字符串处理的一些重要参数
3. hashCode无需再次计算，String创建是hashCode已经计算过了，当作为map中的键时非常方便

### 字符串的不可变性

- 一个string对象在内存(堆)中被创建出来，他就无法被修改
- 可修改的字符串
    - 应该使用StringBuffer 或者 StringBuilder

### JDK 6和JDK 7中substring的原理及区别

- 当调用substring方法的时候，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset 的值是不同的，可能导致内存泄露。
- 在JDK 6中，一般用x.substring(x, y) + ""来解决该问题，原理其实就是生成一个新的字符串并引用他

### JDK 7 中的substring

- 在jdk 7 中，substring方法会在堆内存中创建一个新的数组
- 1.7解决本质：使用`new String`创建了一个新字符串，避免对老字符串的引用。

### String对“+”的重载

- String s = "a" + "b"，编译器会进行常量折叠(因为两个都是编译期常量，编译期可知)，即变成 String s = "ab"
- 对于能够进行优化的(String s = "a" + 变量 等)用 StringBuilder 的 append() 方法替代，最后调用 toString() 方法 (底层就是一个 new String())

### 字符串拼接的几种方式和区别

- +拼接

    - 是将String转成了StringBuilder后，使用其append方法进行处理的

        - 代码检测

            ```java
            String s = "123";
            System.out.println(s+"333");
            // 反编译
            String s = "123";
            System.out.println((new StringBuilder()).append(s).append("333").toString());
            ```

    - **Java是不支持运算符重载的**

- **concat**

    - 创建了一个字符数组，长度是已有字符串和待拼接字符串的长度之和，再把两个字符串的值复制到新的字符数组中，并使用这个字符数组创建一个新的String对象并返回

- **StringBuffer**， append(）

    - 未固定长度，append(）方法修改

        ```java
        //源码
        public synchronized StringBuffer append(String str) {
            toStringCache = null;
            super.append(str);
            return this;
        }
        ```

- **StringBuilder**， append(）

    - 未固定长度，append(）方法修改

        ```java
        //源码
        @Override
        public StringBuilder append(String str) {
            super.append(str);
            return this;
        }
        ```

- 扩展包StringUtils.join

    - StringUtils.join更擅长处理字符串数组或者列表的拼接。

- 效率

    - ```java
        StringBuilder < StringBuffer < concat < + < StringUtils.join
        ```

    - +拼接字符串

        - `for`循环中，每次都是`new`了一个`StringBuilder`，然后再把`String`转成`StringBuilder`，再进行`append`。

- 结论

    - 不是在循环体中进行字符串拼接的话，直接使用`+`
    - 循环体内，字符串的连接方式，使用 `StringBuilder` 的 `append` 方法进行扩展。而不要使用`+`。
    - 并发环境用StringBuffer

### String.valueOf和Integer.toString的区别

- String.valueOf(i)通过调用Integer.toString(i)
    - String i1 = "" + i;
    - 反编译结果:String i1 = (new StringBuilder()).append(i).toString();
    - 首先创建一个StringBuilder对象，然后再调用append方法，再调用toString方法。

### switch对String的支持

> **switch中只能使用整型**

- jdk 1.7 switch支持这样几种数据类型：`byte` `short` `int` `char` `String`
- switch
    - switch对int的判断是直接比较整数的值
    - 对char类型进行比较的时候，实际上比较的是ascii码
    - 字符串的switch是通过`equals()`和`hashCode()`方法来实现的
- 字符串池
- 常量池（运行时常量池、Class常量池）
- intern

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
    //解果
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

- 包装对象的数值比较，不能简单的使用`==`
    - 虽然-128到127之间的数字可以，但是这个范围之外还是需要使用`equals`比较。
- 空指针异常
    - 前面提到，有些场景会进行自动拆装箱，同时也说过，由于自动拆箱，如果包装类对象为null，那么自动拆箱时就有可能抛出NPE。
- 大量拆箱浪费资源
    - 如果一个for循环中有大量拆装箱操作，会浪费很多资源

#### Integer的缓存机制

- 创建对象之前先从IntegerCache.cache中寻找。如果没找到才使用new新建对象。
- 最大值127可以通过`-XX:AutoBoxCacheMax=size`修改
- 缓存通过一个for循环实现。从低到高并创建尽可能多的整数并存储在一个整数数组中。在Integer类第一次被使用的时候被初始化

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

### 如何正确定义接口的返回值

Boolean和boolean类型的isSuccess和success

- idea中getter和setter方法生成

    - boolean: isXXX(),getXXX()
    - Boolean: setXXX(),getXXX()

- Java Bean规定getter,setter

    - 普通参数

        ```java
        public <PropertyType> get<PropertyName>();
        public void set<PropertyName>(<PropertyType> a);
        ```

    - 布尔类型的变量

        ```java
        public boolean is<PropertyName>();
        public void set<PropertyName>(boolean m);
        ```

        

- 另外在反序列化中

    - 在fastjson和jackson的结果中，原来类中的isSuccess字段被序列化成success
    - Gson并不是这么做的，他是通过反射遍历该类中的所有属性，并把其值序列化成json:{"isSuccess":true}
    - 基于以上情况，若采用不同序列化手段，查找变量isSuccess时可能会发生大问题

- 结论

    - **不要使用isSuccess这种形式，而要直接使用success**
    - 阿里巴巴Java开发手册建议使用封装类来定义POJO和RPC返回值中的变量。但是这不意味着可以随意的使用null，我们还是要尽量避免出现对null