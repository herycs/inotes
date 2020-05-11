# Java基础知识

## Java特性

### 面向对象与面向过程

- 面向对象：问题分解为基础步骤，每一步由函数实现，依次调用完成任务
- 面向过程：问题分解为基础步骤，将基础步骤抽象化交由一个个对象完成，各个对象的组合完成任务
- 区别：
    - 面向对象：第三人称，统筹计划，职责内化
    - 面向过程：第一人称，预测，按部就班
- 二者对比：
    - 面向对象占用资源高，速度慢
    - 面向过程占用资源少，速度快

### 面向对象三大特征

继承

> 一个类（继承关系中子类或派生类）继承了另外一个类（继承关系中成为基类或父类）

- 效果：
    - 子类具备父类不曾隐藏起来的可被继承的所有能力
- 实现方式：
    - 继承：继承类
        - 直接拥有继承到的父类的技能
    - 组合：实现接口
        - 受接口的约束，可在继承的方法体中自定义自己的处理策略
- 内存布局
    - 对象体只保存对象参数+虚拟函数表（Vtable）指针
    - Vtable(虚拟函数表)为方法的指针数组

封装

- 情景：一个类将当前具有的特性进行一定程度保护，但其他类可采用当前类允许的方式执行一定操作
- 外部可使用类提供的方法多态

多态

- 同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果
- 必要条件：
    - 有类继承或接口实现
    - 子类要重写父类的方法
    - 父类的引用指针子类的对象（就是用父类对象接受子类对象）

### 面向对象指导思想

可维护性

可复用性

可扩展性

灵活性

### 面向对象五大原则

单一职责原则（SRP）

- 只做一件事
- 高内聚，低耦合

开放封闭原则（OCP）

- 可开扩展不可修改
- 抽象化，多态是开闭原则关键

Liskov(里氏代换原则)(LSP)

- 出现父类的地方一定可以用子类替换（开闭原则的补充，对开闭原则实现抽象化的具体步骤规范）

依赖倒置原则(LOD)

- 依赖于抽象

接口隔离原则(ISP)

- 类之间的依赖应该建立在最小的接口上

> ARTHUR J.RIEL在《 OOD启示录》中所说的：“你并必须遵守这些原则，违背而且你也把这些原则看做警铃，若违背了其中的一条，那么警铃就会响起。”

### Java中的有关概念

#### 重载与重写

重载

- 就是函数或者方法有同样的名称，但是参数列表不相同的样子
- 条件：
    - 方法签名必须有所差异（参数列表，返回类型，权限修饰符，异常）

重写

- 子类方法覆盖父类方法
- 重写条件：
    - 参数列表一样
    - 返回类型一样
    - 访问级别<=被重写访问等级
    - 不抛出新的检查异常，或抛出范围大于父类的异常
- 不可重写情况
    - final标识的方法不可被重写
    - 若不能继承此方法，则不可重写此方法

#### Java的继承与实现

继承

- 多个类具有共同的部分，抽取出抽象类

实现

- 多个类处理方式不同，但目标相同，提取接口

#### Java的继承与组合

组合

- 优点：破坏封装，松耦合，相对独立，扩展性好，动态组合，整体类可对局部类接口进行封装
- 缺点：
    - 整体类不可自动获得局部类的接口
    - 创建整体类对象时需创建所有局部类对象

继承：

- 缺点：破坏封装，紧耦合，独立性差，支持扩展（同时增加了复杂度），不支持动态继承（运行时子类不可选择不同父类），子类不可改变父类接口
- 优点：
    - 子类可自动继承父类接口
    - 创建对象无需创建父类对象

- 建议
    - 同等情况下，优先采用组合

#### 构造函数（构造器）

- 构造器没有返回类型，不会被继承，并且可以有范围修饰符。
- 构造器的函数名称必须和它所属的类的名称相同

#### 类变量、成员变量和局部变量

| 变量类型 | 类变量      | 成员变量 | 局部变量 |
| -------- | ----------- | -------- | -------- |
| 内存位置 | JVM的方法区 | 堆内存   | 栈内存   |

#### 成员变量和方法作用域

- `public` ：该成员变量或方法是对所有类或对象都是可见的，所有类或者对象都可以直接访问
- `private` ：该该成员变量或方法是专有的，只有现代类可以提供访问权限，其他其他或者对象都没有访问权限。子类也没有访问权限。
- `protected` ：某些成员变量或方法对类本身，与同在一个包中的其他类可见，其他包下的类不可访问，除非是他的子类
- `default` ：该成员变量或方法只有自己和其位于同一个包的内可见，其他包内的类不能访问，甚至是它的子类

#### 值传递

值传递、引用传递

为什么说Java中只有值传递

- 编程语言中需要进行方法间的参数传递，这个传递的策略称为求值策略
- 值传递和引用传递最大的区别是传递的过程中有没有复制出一个副本来，如果是传递副本，那就是值传递，否则就是引用传递。
- Java实际上是通过**值传递**实现的参数传递，只不过对于Java对象的传递，传递的内容是对象的引用

### 平台无关性

> 基础：JVM的平台有关性

![image-20200419154812395](D:\Wp\wpMD\images\JVM\java文件编译过程.png)

平台无关性的其他支持

- Java语言规范
    - 通过规定的Java语言中基本数据类型的取值范围和行为
- 类文件
    - 所有Java文件要编译成统一的Class文件
- Java虚拟机
    - 通过Java虚拟机将Class文件转成对应平台的二进制文件等

### 语言无关性

- 实质上Java和JVM虚拟机并非是绑定的，除规范的Java文件编译后的class文件外，JVM认识所有符合JVM字节码规范的文件

## 基础数据类型

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

## 自动拆装箱

- Java中另外一种基本数据类型

    - Java中还存在另外一种基本类型`void`，它也有对应的包装类 `java.lang.Void`

- 包装类型

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

## String

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

## 枚举

### 枚举的用法

- 定义常量

    ```java
    public static enum course{
        math, java, os
    }
    ```

- switch中使用

    ```java
    switch (o){
        case math:
            return course.math;
        case java:
            return course.java;
        case os:
            return course.os;
    }
    ```

- 枚举中添加方法

- 覆盖枚举方法

- 实现接口

- 使用接口组织枚举

### 枚举的实现

- 源代码

    ```java
    public enum EnumDemo2 {
        Java, OS, Math
    }
    ```

- 编译后

    ```java
    public final class EnumDemo2 extends Enum
    {
    
        public static final EnumDemo2 Java;
        public static final EnumDemo2 OS;
        public static final EnumDemo2 Math;
        private static final EnumDemo2 $VALUES[];
    
        public static EnumDemo2[] values()
        {
            return (EnumDemo2[])$VALUES.clone();
        }
    
        public static EnumDemo2 valueOf(String s)
        {
            return (EnumDemo2)Enum.valueOf(com/w/JavaBase/SourceCode/EnumDemo2, s);
        }
    
        private EnumDemo2(String s, int i)
        {
            super(s, i);
        }
    
        static 
        {
            Java = new EnumDemo2("Java", 0);
            OS = new EnumDemo2("OS", 1);
            Math = new EnumDemo2("Math", 2);
            $VALUES = (new EnumDemo2[] {
                Java, OS, Math
            });
        }
    }
    ```

- 对比我们可以发现，enum修饰的类，编译后会成为一个集成Enum类的final类

### 枚举与单例

- 枚举实现的单例在序列化之后被反序列化不会破坏单例。

### Enum类

- 所有枚举类都默认是Enum类的**final**类型的子类

### Java枚举如何比较

- java 枚举值比较用 == 和 equals 方法没啥区别

### switch对枚举的支持

- 所以实质还是 int 参数类型

### 枚举的序列化如何实现

- 在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过`java.lang.Enum`的`valueOf`方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了`writeObject`、`readObject`、`readObjectNoData`、`writeReplace`和`readResolve`等方法。
- 枚举反序列化是并不是通过反射实现，因此不会发生由于反序列化导致的单例破坏问题。

### 枚举的线程安全性问题

- 枚举类型定义的类，在编译后，类被定义为public final T extends Enum，枚举项定义为static类型，在类被加载后初始化，而显然，这个过程交给了虚拟机

    > ​	Java类的加载和初始化过程都是线程安全的
    >
    > - 虚拟机在加载枚举的类会使用ClassLoader的loadClass方法，而这个方法使用同步代码块保证了线程安全

## Java中各种关键字

### transient

### instanceof

### volatile

### synchronized

### final

### static

### const

## 集合类

### Collection和Collections的区别

- Collection 是一个集合接口
- Collections 是一个包装类。 它包含有各种有关集合操作的静态多态方法，不能实例化。

常用集合类的使用

### Set和List区别

- List,Set都是继承自Collection接口。都是用来存储一组相同**类型**的元素的。

    - List特点：有顺序后插法，元素可重复 。

    - Set特点：元素无放入顺序，元素不可重复。
        - set在元素插入时是要有一定的方法来判断元素是否重复的

### ArrayList和LinkedList和Vector的区别

- 都实现了List 接口，使用方式也很相似,主要区别在于因为实现方式的不同,所以对不同的操作具有不同的效率。

> 大数据量情况下

- ArrayList 是一个可改变大小的数组

- LinkedList 是一个双链表
    - LinkedList 还实现了 Queue 接口，List提供了更多的方法,包括 offer(),peek(),poll()等
- Vector是强同步类，但(thread-safe,没有在多个线程之间共享同一个集合/对象)情况下，采用ArryList是更好的选择
    - Vector每次增加一倍
    - ArrayList每次增加一半

### SynchronizedList和Vector的区别

- Vector是java.util包中的一个类。 SynchronizedList是java.util.Collections中的一个静态内部类。
- 多线程的场景中可以直接使用Vector类，也可以使用Collections.synchronizedList(List list)方法来返回一个线程安全的List
- Add方法
    - **1.Vector使用同步方法实现，synchronizedList使用同步代码块实现。 **
    - **2.两者的扩充数组容量方式不一样（两者的add方法在扩容方面的差别也就是ArrayList和Vector的差别。**
- SynchronizedList里面实现的方法几乎都是使用同步代码块包上List的方法。
    - 其中有listIterator和listIterator(int index)并没有做同步处理
    - 注意点：
        - 在使用SynchronizedList进行遍历的时候要手动加锁
- Vector基本使用同步方法实现
- 区别分析
    - 数据增加
        - Vector更合适，可指定初始化大小
    - 代码块和方法
        - 同步代码块在锁定的范围上可能比同步方法要小一般来讲大小和方法成反比
        - 同步块可以更加精确的控制锁的作用域（锁的作用域就是从锁被获取到其被释放的时间），同步方法的锁的作用域就是整个方法
        - 静态代码块可以选择对哪个对象加锁，但是静态方法只能给this对象加锁

### Set如何保证元素不重复

- 实现方式不同主要分为两大类。HashSet和TreeSet
    - 1.TreeSet 是二差树实现的,Treeset中的数据是自动排好序的，不允许放入null值 
    - 2.HashSet 是哈希表实现的,HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不重复，就如数据库中唯一约束
- 细节
    - 在HashSet中，基本的操作都是有HashMap底层实现的，因为HashSet底层是用HashMap存储数据的。当向HashSet中添加元素的时候，首先计算元素的hashcode值，然后通过扰动计算和按位与的方式计算出这个元素的存储位置，如果这个位置位空，就将元素添加进去；如果不为空，则用equals方法比较元素是否相等，相等就不添加，否则找一个空位添加。
    - TreeSet的底层是TreeMap的keySet()，而TreeMap是基于红黑树实现的，红黑树是一种平衡二叉查找树，它能保证任何一个节点的左右子树的高度差不会超过较矮的那棵的一倍。
- 检查重复
    - TreeMap是按key排序的，元素在插入TreeSet时compareTo()方法要被调用，所以TreeSet中的元素要实现Comparable接口。
    - TreeSet作为一种Set，它不允许出现重复元素。TreeSet是用compareTo()来判断重复元素的。

### HashMap、HashTable、ConcurrentHashMap区别

- Map&Table
    - 线程安全：
        - HashTable 中的方法是同步的，而HashMap中的方法在默认情况下是非同步的。
        - 在多线程并发的环境下，可以直接使用HashTable，但是要使用HashMap的话就要自己增加同步处理了。
    - 继承关系：
        - HashTable是基于陈旧的Dictionary类继承来的。 
        - HashMap继承的抽象类AbstractMap实现了Map接口
    - null
        - HashTable中，key和value都不允许出现null值，否则会抛出NullPointerException异常。
        - HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。
    - 初始值
        - HashTable中的hash数组初始大小是11，增加的方式是 old*2+1。
        - HashMap中hash数组的默认大小是16，而且一定是2的指数。
    - 哈希值的使用不同 ： 
        - HashTable直接使用对象的hashCode。
        - HashMap重新计算hash值。
    - 遍历
        - 遍历方式的内部实现上不同 ： Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。 
        - HashMap 实现 Iterator，支持fast-fail，Hashtable的 Iterator 遍历支持fast-fail，用 Enumeration 不支持 fast-fail
- HashMap&CurrentHashMap
    - 遍历方式的内部实现上不同 ：桶数组 
        - Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。
        - HashMap 实现 Iterator，支持fast-fail，Hashtable的 Iterator 遍历支持fast-fail，用 Enumeration 不支持 fast-fail
    - 锁
        - ConcurrentHashMap在每一个分段上都用锁进行了保护。
        - HashMap没有锁机制。

Java 8中Map相关的红黑树的引用背景、原理等

HashMap的容量、扩容、hash等原理

### Java 8中stream相关用法

- Stream有以下特性及优点：
    - 无存储。Stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
    - 为函数式编程而生。对Stream的任何修改都不会修改背后的数据源，比如对Stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新Stream。
    - 惰式执行。Stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
    - 可消费性。Stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。
- 最终操作会消耗流，产生一个最终结果。
    - 在最终操作之后，不能再次使用流，也不能在使用任何中间操作，否则将抛出异常
- 操作类型
    - Stream的创建有两种方式，分别是通过集合类的stream方法、通过Stream的of方法。
    - Stream的中间操作可以用来处理Stream，中间操作的输入和输出都是Stream，中间操作可以是过滤、转换、排序等。
    - Stream的最终操作可以将Stream转成其他形式，如计算出流中元素的个数、将流装换成集合、以及元素的遍历等。

Apache集合处理工具类的使用

不同版本的JDK中HashMap的实现的区别以及原因

### Arrays.asList获得的List使用时需要注意什么

- asList 得到的只是一个 Arrays 的内部类，一个原来数组的视图 List，因此如果对它进行增删操作会报错
- 用 ArrayList 的构造器可以将其转变成真正的 ArrayLi

Collection如何迭代

#### Enumeration和Iterator区别

- 函数接口不同
    - ​    Enumeration只有2个函数接口。通过Enumeration，我们只能读取集合的数据，而不能对数据进行修改。

    - Iterator只有3个函数接口。Iterator除了能读取集合的数据之外，也能数据进行删除操作。
- Iterator支持fail-fast机制，而Enumeration不支持
    - Enumeration 是JDK 1.0添加的接口。使用到它的函数包括Vector、Hashtable等类，这些类都是JDK 1.0中加入的，Enumeration存在的目的就是为它们提供遍历接口。Enumeration本身并没有支持同步，而在Vector、Hashtable实现Enumeration时，添加了同步。
    - 而Iterator 是JDK 1.2才添加的接口，它也是为了HashMap、ArrayList等集合提供遍历接口。Iterator是支持fail-fast机制的：当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

如何在遍历的同时删除ArrayList中的元素

### fail-fast 和 fail-safe

- fail-fast

    - 立即报告任何可能表明故障的情况的系统

- fail-safe

    - 为了避免触发fail-fast机制，导致异常，我们可以使用Java中提供的一些采用了fail-safe机制的集合类。

        这样的集合容器在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

- fail-safe集合的所有对集合的修改都是先拷贝一份副本，然后在副本集合上进行的，并不是直接对原集合进行修改。并且这些修改方法，如add/remove都是通过加锁来控制并发的。

- 所以，CopyOnWriteArrayList中的迭代器在迭代的过程中不需要做fail-fast的并发检测。（因为fail-fast的主要目的就是识别并发，然后通过异常的方式通知用户）

CopyOnWriteArrayList

ConcurrentSkipListMap

## IO

### 字符流、字节流

- 字节，字符
    - Bit最小的二进制单位 ，是计算机的操作部分。取值0或者1
    - Byte（字节）是计算机操作数据的最小单位由8位bit组成 取值（-128-127）
    - Char（字符）是用户的可读写的最小单位，在Java里面由16位bit组成 取值（0-65535）
- 字节流
    - 操作byte类型数据，主要操作类是OutputStream、InputStream的子类；不用缓冲区，直接对文件本身操作。
- 字符流
    - 操作字符类型数据，主要操作类是Reader、Writer的子类；使用缓冲区缓冲字符，不关闭流就不会输出任何内容。

输入流、输出流

### 字节流和字符流之间的相互转换

- OutputStreamWriter：是Writer的子类，将输出的字符流变为字节流，即将一个字符流的输出对象变为字节流输出对象。
- InputStreamReader：是Reader的子类，将输入的字节流变为字符流，即将一个字节流的输入对象变为字符流的输入对象

### 同步、异步

- 描述背调方

### 阻塞、非阻塞

- 描述调用方

### Linux 5种IO模型

- 阻塞IO

    - 当用户线程发出IO请求之后，内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出CPU。当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除block状态。

- 非阻塞IO

    - 当用户线程发起一个read操作后，并不需要等待，而是马上就得到了一个结果。如果结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。

- 多路复用IO

    - 在Java NIO中，是通过selector.select()去查询每个通道是否有到达事件，如果没有事件，则一直阻塞在那里，因此这种方式会导致用户线程的阻塞。
    - 只需要使用一个线程就可以管理多个socket
        - 不必维护这些线程和进程，并且只有在真正有socket读写事件进行时，才会使用IO资源，所以它大大减少了资源占用。

- 信号驱动IO

    > 类似食堂吃饭，用户点餐，食堂准备好后通知用户取餐

    - 在信号驱动IO模型中，当用户线程发起一个IO请求操作，会给对应的socket注册一个信号函数，然后用户线程会继续执行。
    - 当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用IO读写操作来进行实际的IO请求操作。

- 异步IO模型

    > 类似餐厅吃饭，用户点餐，之后餐厅会将食物呈现到用户桌子上
    >
    > - 异步IO是需要操作系统的底层支持，在Java 7中，提供了Asynchronous IO。

    - 异步IO模型中，IO操作的两个阶段都不会阻塞用户线程，这两个阶段都是由内核自动完成，然后发送一个信号告知用户线程操作已完成。用户线程中不需要再次调用IO函数进行具体的读写。

### BIO、NIO和AIO的区别

- IO	

    - 在 Java 编程中，直到最近一直使用 流 的方式完成 I/O。所有 I/O 都被视为单个的字节的移动，通过一个称为 Stream 的对象一次移动一个字节。流 I/O 用于与外部世界接触。它也在内部使用，用于将对象转换为字节，然后再转换回对象。

- BIO

    - Java BIO即Block I/O ， 同步并阻塞的IO。

- NIO

    > 与基础IO区别在于数据打包和传输的方式

    - 原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。
    - 面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。

- AIO

    - Java AIO即Async非阻塞，是异步非阻塞的IO。

- 区别&联系

    - BIO （Blocking I/O）：同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。
    - NIO （New I/O）：同时支持阻塞与非阻塞模式，但这里我们以其同步非阻塞I/O模式来说明，那么什么叫做同步非阻塞？如果还拿烧开水来说，NIO的做法是叫一个线程不断的轮询每个水壶的状态，看看是否有水壶的状态发生了改变，从而进行下一步的操作。
    - AIO （ Asynchronous I/O）：异步非阻塞I/O模型。异步非阻塞与同步非阻塞的区别在哪里？异步非阻塞无需一个线程去轮询所有IO操作的状态改变，在相应的状态改变后，系统会通知对应的线程来处理。

- 适用场景

    - BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
    - NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
    - AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

## 反射

### 什么是反射

- 反射机制指的是程序在运行时能够获取自身的信息。

### 反射有什么作用

- 在运行时判断任意一个对象所属的类。

- 在运行时判断任意一个类所具有的成员变量和方法。

- 在运行时任意调用一个对象的方法

- 在运行时构造任意一个类的对象

### Class类

- Java的Class类是java反射机制的基础,通过Class类我们可以获得关于一个类的相关信息
- Java.lang.Class是一个比较特殊的类，它用于封装被装入到JVM中的类（包括类和接口）的信息。当一个类或接口被装入的JVM时便会产生一个与之关联的java.lang.Class对象，可以通过这个Class对象对被装入类的详细信息进行访问。

java.lang.reflect.*

### 静态代理

- 所谓静态代理，就是代理类是由程序员自己编写的，在编译期就确定好了的。

### 动态代理

- 动态代理中的代理类并不要求在编译期就确定，而是可以在运行期动态生成，从而实现对目标对象的代理功能。

#### 动态代理和反射的关系

- 反射是动态代理的一种实现方式。

#### 动态代理的几种实现方式

- JDK动态代理：java.lang.reflect 包中的Proxy类和InvocationHandler接口提供了生成动态代理类的能力。
- Cglib动态代理：Cglib (Code Generation Library )是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。

### AOP

- Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。
- 如果目标类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。

## 序列化

### 什么是序列化与反序列化

- 序列化是将对象转换为可传输格式的过程。 
- 序列化是将对象的状态信息转换为可存储或传输的形式的过程。

### Java如何实现序列化与反序列化

> 在真实的应用场景中，我们需要将这些对象持久化下来，并且能够在需要的时候把对象重新读取出来。Java的对象序列化可以帮助我们实现该功能。

### java中提供的技术支持

> java.io.Serializable
>
> java.io.Externalizable
>
> ObjectOutput
>
> ObjectInput
>
> ObjectOutputStream
>
> ObjectInputStream

#### Serializable接口

> 当试图对一个对象进行序列化的时候，如果遇到不支持 Serializable 接口的对象。在此情况下，将抛出 `NotSerializableException`。
>
> 如果要序列化的类有父类，要想同时将在父类中定义过的变量持久化下来，那么父类也应该集成`java.io.Serializable`接口

- 序列化接口没有方法或字段，仅用于标识可序列化的语义。
- 类通过实现 `java.io.Serializable` 接口以启用其序列化功能。

#### Externalizable接口

> 类中没有无参数的构造函数，在运行时会抛出异常：`java.io.InvalidClassException`

- Externalizable继承了Serializable，该接口中定义了两个抽象方法：`writeExternal()`与`readExternal()`。
- 当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员重写`writeExternal()`与`readExternal()`方法。
- 在使用Externalizable进行序列化的时候，在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。
- 注意：
    - 实现Externalizable接口的类必须要提供一个public的无参的构造器。

#### Serializable 和 Externalizable 有何不同

- 对⼀个对象进⾏序列化的时候， 如果遇到不⽀持`Serializable` 接口的对象。 在此情况下， 将抛`NotSerializableException`。
- 如果要序列化的类有⽗类， 要想同时将在⽗类中定义过的变量持久化下来， 那么⽗类也应该集成`java.io.Serializable`接口。
- Externalizable`继承了`Serializable
    - 该接⼜中定义了两个抽象⽅法：`writeExternal()`与`readExternal()`。
    - 当使⽤`Externalizable`接口来进⾏序列化与反序列化的时候需要重写writeExternal()与readExternal()⽅法。
        - 如果没有在这两个⽅法中定义序列化实现细节， 那么序列化之后， 对象内容为空。
        - 实现`Externalizable`接⼜的类必须要提供⼀个`public`的⽆参的构造器。
    - 结论：实现`Externalizable`， 并实现`writeExternal()`和`readExternal()`⽅法可以指定序列化哪些属性。

为什么需要序列化

#### serialVersionUID

- 序列化ID
    - 虚拟机是否允许反序列化， 不仅取决于类路径和功能代码是否⼀致， ⼀个⾮常重要的⼀点是两个类的序列化 ID 是否⼀致
    - JVM会把传来的字节流中的`serialVersionUID`与本地相应实体类的`serialVersionUID`进⾏⽐较
        -  如果相同就认为是⼀致的， 可以进⾏反序列化
        -  否则就会出现序列化版本不⼀致的异常， 即是`InvalidCastException`。
    - 当实现`java.io.Serializable`接口的类**没有**显式地定义⼀个`serialVersionUID`变量时候， Java序列化机制会根据编译的Class⾃动⽣成⼀个`serialVersionUID`作序列化版本⽐较⽤
        - 如果Class⽂件没有发⽣变化， 就算再编译多 次， serialVersionUID也不会变化的。
        - 如果发⽣了变化，那么这个⽂件对应的`serialVersionUID`也就会发⽣变化。
    - 基于以上情况，序列化后更改类，而后反序列化会报错。

#### 为什么serialVersionUID不能随便改

- 没有实现这个接口，想要被序列化的话，就会抛出`java.io.NotSerializableException`异常。

- 进行序列化操作时，会判断要被序列化的类是否是`Enum`、`Array`和`Serializable`类型，如果都不是则直接抛出`NotSerializableException`。

- 序列化策略：

    - 在序列化过程中，如果被序列化的类中定义了`writeObject` 和 `readObject` 方法，虚拟机会试图调用对象类里的 `writeObject` 和 `readObject` 方法，进行用户自定义的序列化和反序列化。
    - 如果没有这样的方法，则默认调用是 `ObjectOutputStream` 的 `defaultWriteObject` 方法以及 `ObjectInputStream` 的 `defaultReadObject` 方法。
    - 用户自定义的 `writeObject` 和 `readObject` 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。
        - 对于一些特殊字段需要定义序列化的策略的时候，可以考虑使用transient修饰，并自己重写`writeObject` 和 `readObject` 方法，如`java.util.ArrayList`中就有这样的实现。

- 底层实现：

    - 对`serialVersionUID`做了比较，如果发现不相等，则直接抛出异常。

        深入看一下`getSerialVersionUID`方法：

        ```
        public long getSerialVersionUID() {
            if (suid == null) {
                suid = AccessController.doPrivileged(
                    new PrivilegedAction<Long>() {
                        public Long run() {
                            return computeDefaultSUID(cl);
                        }
                    }
                );
            }
            return suid.longValue();
        }
        ```

    - 在没有定义`serialVersionUID`的时候，会调用`computeDefaultSUID` 方法，生成一个默认的`serialVersionUID`

#### transient

- `transient` 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以**阻止该变量被序列**化到文件中
- 在被反序列化后，`transient` 变量的值被设为初始值。

#### 序列化底层原理

- 对象序列化保存的是对象的"状态"，即它的成员变量。由此可知，**对象序列化不会关注类中的静态变量**。

- 在Java中，只要一个类实现了`java.io.Serializable`接口

- 序列化与反序列化

    - 1、在Java中，只要一个类实现了`java.io.Serializable`接口，那么它就可以被序列化。

    - 2、通过`ObjectOutputStream`和`ObjectInputStream`对对象进行序列化及反序列化

    - 3、虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 `private static final long serialVersionUID`）

    - 4、序列化并不保存静态变量。

    - 5、要想将父类对象也序列化，就需要让父类也实现`Serializable` 接口。

    - 6、Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文

        中，在被反序列化后，transient 变量的值被设为初始值

    - 7、服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。

- ArrayList序列化

    - ArrayList中elementData是`transient`的，本不能被反序列化出来，但实际结果并非如此

    - 原因是其提供了方法：

        - ArrayList中定义了来个方法： `writeObject`和`readObject`。
        - 序列化的过程是：
            - 在序列化过程中，如果被序列化的类中定义了writeObject 和 readObject 方法，虚拟机会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化。
            - 如果没有这样的方法，则默认调用是 ObjectOutputStream 的 defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject 方法。
            - 用户自定义的 writeObject 和 readObject 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。

    - 为了避免elementData无效数据被序列化

        - ArrayList实际上是动态数组，每次在放满以后自动增长设定的长度值，如果数组自动增长长度设为100，而实际只放了一个元素，那就会序列化99个null元素。为了保证在序列化的时候不会将这么多null同时进行序列化，ArrayList把元素数组设置为transient。

    - 为了保存elementData中有效数据

        - `writeObject`方法把`elementData`数组中的元素遍历的保存到输出流（ObjectOutputStream）中。
        - `readObject`方法从输入流（ObjectInputStream）中读出对象并保存赋值到`elementData`数组中。

    - 对象的序列化过程通过ObjectOutputStream和ObjectInputputStream来实现的

    - ObjectOutputStream的writeObject的调用栈：

        ```less
        writeObject ---> writeObject0 --->writeOrdinaryObject--->writeSerialData--->invokeWriteObject
        ```

- 一个类中包含writeObject 和 readObject 方法调用方式

    - 在使用ObjectOutputStream的writeObject方法和ObjectInputStream的readObject方法时，会通过反射的方式调用。

- ObjectOutputStream的writeObject的调用栈：

    ```less
    writeObject ---> writeObject0 --->writeOrdinaryObject--->writeSerialData--->invokeWriteObject
    ```

#### 序列化如何破坏单例模式

- 普通的Java类的反序列化过程中，会通过反射调用类的默认构造函数来初始化对象。所以，即使单例中构造函数是私有的，也会被反射给破坏掉。由于反序列化后的对象是重新new出来的，所以这就破坏了单例。

#### protobuf

- Protocol Buffer (简称Protobuf) 是Google出品的性能优异、跨语言、跨平台的序列化库。

### 为什么说序列化并不安全

## 注解

### 元注解

- 定义其他注解的注解
- 元注解有四个:
    - @Target（表示该注解可以用于什么地方）
    - @Retention（表示再什么级别保存该注解信息）
    - @Documented（将此注解包含再javadoc中）
    - @Inherited（允许子类继承父类中的注解）。

### 自定义注解

- 在Java中，类使用class定义，接口使用interface定义，注解和接口的定义差不多，增加了一个@符号，即@interface

Java中常用注解使用

注解与反射的结合

如何自定义一个注解？

## 泛型

### 什么是泛型

- Java泛型（ generics） 是JDK 5中引⼊的⼀个新特性， 允许在定义类和接⼜的时候使⽤类型参数（ type parameter）

### 类型擦除

- 编译器处理泛型
    - 1.`Code specialization`。在实例化一个泛型类或泛型方法时都产生一份新的目标代码（字节码or二进制代码）。例如，针对一个泛型`list`，可能需要 针对`string`，`integer`，`float`产生三份目标代码。
    - 2.`Code sharing`。对每个泛型类只生成唯一的一份目标代码；该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。
- 类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码上。
    - 编译器只为泛型类型生成一份字节码，并将其实例关联到这份字节码上。
    - 类型擦除的关键在于从泛型类型中清除类型参数的相关信息，并且再必要的时候添加类型检查和类型转换的方法。
    - 类型擦除可以简单的理解为将泛型java代码转换为普通java代码，只不过编译器更直接点，将泛型java代码直接转换成普通java字节码。
    - 类型擦除的主要过程如下：
        -  1.将所有的泛型参数用其最左边界（最顶级的父类型）类型替换。
        -  2.移除所有的类型参数。

### 泛型带来的问题

- 泛型遇到重载
    - 参数`List`和`List`编译之后都被擦除了，变成了一样的原生类型List，擦除动作导致这两个方法的特征签名变得一模一样。
- 泛型遇到catch
    - 自定义了一个泛型异常类GenericException，不应该尝试用多个catch取匹配不同的异常类型
- 泛型内包含静态变量
    - 由于经过类型擦除，所有的泛型类实例都关联到同一份字节码上，泛型类的所有静态变量是共享的。
- 总结
    - 1.虚拟机中没有泛型，只有普通类和普通方法,所有泛型类的类型参数在编译时都会被擦除,泛型类并没有自己独有的Class类对象。
    - 2.创建泛型对象时请指明类型，让编译器尽早的做参数检查
    - 3.不要忽略编译器的警告信息，那意味着潜在的`ClassCastException`等着你。
    - 4.静态变量是被泛型类的所有实例所共享的。
    - 5.泛型的类型参数不能用在`Java`异常处理的`catch`语句中。

### 泛型中K T V E ？ object等的含义

> E - Element (在集合中使用，因为集合中存放的是元素)
>
> T - Type（Java 类）
>
> K - Key（键）
>
> V - Value（值）
>
> N - Number（数值类型）
>
> ？ - 表示不确定的java类型（无限制通配符类型）
>
> S、U、V - 2nd、3rd、4th types

- Object - 是所有类的根类，任何类的对象都可以设置给该Object引用变量，使用的时候可能需要类型强制转换，但是用使用了泛型T、E等这些标识符后，在实际用之前类型就已经确定了，不需要再进行类型强制转换。

泛型各种用法

#### 限定通配符和非限定通配符

- `限定通配符`对类型进⾏限制， 泛型中有两种限定通配符：

    表示类型的上界，格式为：<？ extends T>，即类型必须为T类型或者T子类 表示类型的下界，格式为：<？ super T>，即类型必须为T类型或者T的父类

- 泛型类型必须⽤限定内的类型来进⾏初始化，否则会导致编译错误。

    `⾮限定通配符`表⽰可以⽤任意泛型类型来替代，类型为

#### 上下界限定符extends 和 super

- <? extends T>和``<? super T>是Java泛型中的“通配符（Wildcards）”和“边界（Bounds）”的概念。

    <? extends T>：是指 “上界通配符（Upper Bounds Wildcards）”，即泛型中的类必须为当前类的子类或当前类。

    <? super T>：是指 “下界通配符（Lower Bounds Wildcards）”，即泛型中的类必须为当前类或者其父类。

### List和原始类型List之间的区别?

- 原始类型List和带参数类型之间的主要区别是，在编译时编译器不会对原始类型进行类型安全检查，却会对带参数的类型进行检查。
- 通过使用Object作为类型，可以告知编译器该方法可以接受任何类型的对象。
- 它们之间的第二点区别是，你可以把任何带参数的类型传递给原始类型List，但却不能把List传递给接受 List的方法，因为会产生编译错误。

### List<?>和List之间的区别是什么?

- List 是一个未知类型的List，而List 其实是任意类型的List

## 异常

### Error和Exception

> 均继承自Throwable类。

- Error表⽰系统级的错误， 是java**运⾏环境内部错误**或者**硬件问题**， 不能指望程序来处理这样的问题， 除了退出运⾏外别⽆选择， 它是**Java虚拟机抛出**的。
- Exception 表⽰程序需要捕捉、 需要处理的常， 是由与**程序设计的不完善**⽽出现的问题， 程序必须处理的问题。

### 异常类型

> 受检异常（ checked exception） 
>
> ⾮受检异常（ unchecked exception）

- 我们在程序中调⽤他的时候， ⼀定要对该异常进⾏处理（ 捕获或者向上抛出） ， 否则是⽆法编译通过的。
- 对于⾮受检异常来说， ⼀般是运⾏时异常， 继承⾃RuntimeException。 在编写代码的时候， 不需要显⽰的捕获，但是如果不捕获， 在运⾏期如果发⽣异常就会中断程序的执⾏。

### 异常相关关键字

> throws、 throw、 try、 catch、 finally
>
> try⽤来指定⼀块预防所有异常的程序；
>
> catch⼦句紧跟在try块后⾯， ⽤来指定你想要捕获的异常的类型；
>
> finally为确保⼀段代码不管发⽣什么异常状况都要被执⾏；
>
> throw语句⽤来明确地抛出⼀个异常；
>
> throws⽤来声明⼀个⽅法可能抛出的各种异常；

### 正确处理异常

- 1、 ⾃⼰处理。 
- 2、 向上抛， 交给调⽤者处理。

### 自定义异常

- 继承`Exception`的⼦类
- 本质：是继承⼀个API标准异常类， ⽤新定义的异常处理信息覆盖原有信息的过程

### 异常链

try-with-resources

- 关闭资源的常用方式就是在finally块里是释放

finally和return的执行顺序

- `try()` ⾥⾯有⼀个`return`语句， 那么后⾯的`finally{}`⾥⾯的code会不会被执⾏
- 如果try中有return语句， 那么finally中的代码还是会执⾏。
    - 因为return表⽰的是要整个⽅法体返回， 所以，finally中的语句会在return之前执⾏。
    - 但是return前执行的finally块内，对数据的修改效果对于引用类型和值类型会不同

## 时间处理

### 时区

冬令时和夏令时

- 夏令时、冬令时的出现，是为了充分利用夏天的日照，所以时钟要往前拨快一小时，冬天再把表往回拨一小时。其中夏令时从3月第二个周日持续到11月第一个周日。

    - 冬令时： 北京和洛杉矶时差：16 北京和纽约时差：13

    - 夏令时： 北京和洛杉矶时差：15 北京和纽约时差：12

### 时间戳

- 时间戳（timestamp），一个能表示一份数据在某个特定时间之前已经存在的、 完整的、 可验证的数据,通常是一个字符序列，唯一地标识某一刻的时间。
- 时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。通俗的讲， 时间戳是一份能够表示一份数据在一个特定时间点已经存在的完整的可验证的数据。

### Java中时间API(Java 8)

格林威治时间

- 自1924年2月5日开始，格林尼治天文台负责每隔一小时向全世界发放调时信息。
- 地球每天的自转是有些不规则的，而且正在缓慢减速，因此格林尼治平时基于天文观测本身的缺陷，已经被原子钟报时的协调世界时（UTC）所取代。
- GMT+8表示中国的时间，是因为中国位于东八区，时间上比格林威治时间快8个小时。

### CET、UTC、GMT、CST几种常见时间的含义和关系

- CET
    - 欧洲中部时间（英語：Central European Time，CET）是比世界标准时间（UTC）早一个小时的时区名称之一。它被大部分欧洲国家和部分北非国家采用。冬季时间为UTC+1，夏季欧洲夏令时为UTC+2。
- UTC
    - 协调世界时，又称世界标准时间或世界协调时间，简称UTC，从英文“Coordinated Universal Time”／法文“Temps Universel Cordonné”而来。台湾采用CNS 7648的《资料元及交换格式–资讯交换–日期及时间的表示法》（与ISO 8601类似）称之为世界统一时间。中国大陆采用ISO 8601-1988的国标《数据元和交换格式信息交换日期和时间表示法》（GB/T 7408）中称之为国际协调时间。协调世界时是以原子时秒长为基础，在时刻上尽量接近于世界时的一种时间计量系统。
- GMT
    - 格林尼治标准时间（旧译格林尼治平均时间或格林威治标准时间；英语：Greenwich Mean Time，GMT）是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。
- CST
    - 北京时间，China Standard Time，又名中国标准时间，是中国的标准时间。在时区划分上，属东八区，比协调世界时早8小时，记为UTC+8，与中华民国国家标准时间（旧称“中原标准时间”）、香港时间和澳门时间和相同。當格林威治時間為凌晨0:00時，中國標準時間剛好為上午8:00。
- 关系
    - CET=UTC/GMT + 1小时 
    - CST=UTC/GMT +8 小时 
    - CST=CET+9

### SimpleDateFormat的线程安全性问题

- SimpleDateFormat是Java提供的一个格式化和解析日期的工具类。它允许进行格式化（日期 -> 文本）、解析（文本 -> 日期）和规范化。

- SimpleDateFormat作为一个**非线程安全**的类，被当做了共享变量在多个线程中进行使用，这就出现了线程安全问题。

- 源码

    - format方法在执行过程中，会使用一个成员变量calendar来保存时间，parse方法不是线程安全的。
    - 故，当定义SimpleDateFormat为一个静态变量时多线程情况下会出现问题

- 解决方法

    > 以下方法本质保证线程操作的SimpleDateFormat不受其他线程影响

    - 使用局部变量：定义SimpleDateFormat为局部变量
    - 加同步锁
    - 使用 ThreadLocal
    - JDK 1.8使用使用DateTimeFormatter

### Java 8中的时间处理

> java.time.*包中
>
> `Instant`： 时间戳
>
> `Duration`： 持续时间， 时间差
>
> `LocalDate`： 只包含⽇期， ⽐如： 2016-10-20
>
> `LocalTime`： 只包含时间， ⽐如： 23![12](https://github.githubassets.com/images/icons/emoji/12.png)10
>
> `LocalDateTime`： 包含⽇期和时间， ⽐如： 2016-10-20 23![14](https://github.githubassets.com/images/icons/emoji/14.png)21
>
> `Period`： 时间段
>
> `ZoneOffset`： 时区偏移量， ⽐如： +8:00
>
> `ZonedDateTime`： 带时区的时间
>
> `Clock`： 时钟， ⽐如获取⽬前美国纽约的时间

- `LocalDate`表⽰⽇期， 年⽉⽇， `LocalTime`表⽰时间， 时分 秒

- 获取当前时间

    - ```java
        LocalDate today = LocalDate.now();
        ```

- 创建指定时间

    - ```java
        LocalDate date = LocalDate.of(2018, 01, 01);
        ```

- 日期间天数差

    - ```java
        Period period = Period.between(LocalDate.of(2018, 1, 5),LocalDate.of(2018, 2, 5));
        ```

        

### 如何在东八区的计算机上获取美国时间

> Java8 中加入了对时区的支持，带时区的时间为分别为：`ZonedDate`、`ZonedTime`、`ZonedDateTime`。
>
> 其中每个时区都对应着 ID，地区ID都为 “{区域}/{城市}”的格式，如`Asia/Shanghai`、`America/Los_Angeles`等。

- ```java
    LocalDateTime now = LocalDateTime.now(ZoneId.of("America/Los_Angeles"));
    System.out.println(now);
    ```

代码无法获取美国时间

- ```java
    System.out.println(Calendar.getInstance(TimeZone.getTimeZone("America/Los_Angeles")).getTime());
    ```

    原因：System.out.println来输出一个时间的时候，他会调用Date类的toString方法，而该方法会读取操作系统的默认时区来进行时间的转换。

    故处理方法是修改本地时区为美国

### yyyy和YYYY有什么区别？

> y表示Year ,而Y表示Week Year

- 不同的国家对于一周的开始和结束的定义是不同的。

    - 中国，我们把星期一作为一周的第一天
    - 美国，他们把星期日作为一周的第一天

- 统一时间

    - 国际标准化组织的国际标准ISO 8601是日期和时间的表示方法，全称为《数据存储和交换形式·信息交换·日期和时间的表示方法》。

        > 在 ISO 8601中。对于一年的第一个日历星期有以下四种等效说法：
        >
        > - 1，本年度第一个星期四所在的星期；
        > - 2，1月4日所在的星期；
        > - 3，本年度第一个至少有4天在同一星期内的星期；
        > - 4，星期一在去年12月29日至今年1月4日以内的星期；

- 结论

    - 我们要表示日期的时候，一定要使用 yyyy-MM-dd 而不是 YYYY-MM-dd ，这两者的返回结果大多数情况下都一样，但是极端情况就会有问题了。

## 编码方式

### 什么是ASCII？

- ASCII（ American Standard Code for InformationInterchange， 美国信息交换标准代码） 是基于拉丁字母的⼀套电脑编码系统， 主要⽤于显⽰现代英语和其他西欧语⾔。

- 现今最通⽤的单字节编码系统， 并等同于国际标准ISO/IEC646。

- 具体

    > 使⽤7 位⼆进制数（ 剩下的1位⼆进制为0） 来表⽰所有的⼤写和⼩写字母， 数字0 到9、 标点符号， 以及在美式英语中使⽤的特殊控制字符。
    >
    > - 0～31及127(共33个)是控制字符或通信专⽤字符（ 其余为可显⽰字符） ， 如控制符： LF（ 换⾏） 、 CR（ 回车） 、 FF（ 换页） 、 DEL（ 删除） 、 BS（ 退格)、 BEL（ 响铃） 等； 通信专⽤字符： SOH（ ⽂头） 、 EOT（ ⽂尾） 、 ACK（ 确认） 等；
    >
    > - ASCII值为8、 9、 10 和13 分别转换为退格、 制表、 换⾏和回车字符。 它们并没有特定的图形显⽰， 但会依不同的应⽤程序，⽽对⽂本显⽰有不同的影响
    >
    > - 32～126(共95个)是字符(32是空格） ， 其中48～57为0到9⼗个阿拉伯数字。
    >
    > - 65～90为26个⼤写英⽂字母， 97～122号为26个⼩写英⽂字母， 其余为⼀些标点符号、 运算符号等。

### Unicode

- Unicode（中文：万国码、国际码、统一码、单一码）是计算机科学领域里的一项业界标准。它对世界上大部分的文字系统进行了整理、编码，使得计算机可以用更为简单的方式来呈现和处理文字。

#### 有了Unicode为啥还需要UTF-8

- Unicode 是字符集。UTF-8 是编码规则。
- unicode虽然统一了全世界字符的二进制编码，但没有规定如何存储。
- UTF = Unicode Tranformation Format
- UTF-8使用可变长度字节来储存 Unicode字符，例如ASCII字母继续使用1字节储存，重音文字、希腊字母或西里尔字母等使用2字节来储存，而常用的汉字就要使用3字节。辅助平面字符则使用4字节。

### UTF8、UTF16、UTF32区别

- UTF-8 使用一至四个字节为每个字符编码，其中大部分汉字采用三个字节编码，少量不常用汉字采用四个字节编码。因为 UTF-8 是可变长度的编码方式，相对于 Unicode 编码可以减少存储占用的空间，所以被广泛使用。
- UTF-16 使用二或四个字节为每个字符编码，其中大部分汉字采用两个字节编码，少量不常用汉字采用四个字节编码。UTF-16 编码有大尾序和小尾序之别，即 UTF-16BE 和 UTF-16LE，在编码前会放置一个 U+FEFF 或 U+FFFE（UTF-16BE 以 FEFF 代表，UTF-16LE 以 FFFE 代表），其中 U+FEFF 字符在 Unicode 中代表的意义是 ZERO WIDTH NO-BREAK SPACE，顾名思义，它是个没有宽度也没有断字的空白。
- UTF-32 使用四个字节为每个字符编码，使得 UTF-32 占用空间通常会是其它编码的二到四倍。UTF-32 与 UTF-16 一样有大尾序和小尾序之别，编码前会放置 U+0000FEFF 或 U+0000FFFE 以区分。

#### 有了UTF8为什么还需要GBK？

### GBK、GB2312、GB18030之间的区别

- GB2312（1980年）：16位字符集，收录有6763个简体汉字，682个符号，共7445个字符； 优点：适用于简体中文环境，属于中国国家标准，通行于大陆，新加坡等地也使用此编码； 缺点：不兼容繁体中文，其汉字集合过少。
- GBK（1995年）：16位字符集，收录有21003个汉字，883个符号，共21886个字符； 优点：适用于简繁中文共存的环境，为简体Windows所使用（代码页cp936），向下完全兼容gb2312，向上支持 ISO-10646 国际标准 ；所有字符都可以一对一映射到unicode2.0上； 缺点：不属于官方标准，和big5之间需要转换；很多搜索引擎都不能很好地支持GBK汉字。
- GB18030（2000年）：32位字符集；收录了27484个汉字，同时收录了藏文、蒙文、维吾尔文等主要的少数民族文字。 优点：可以收录所有你能想到的文字和符号，属于中国最新的国家标准； 缺点：目前支持它的软件较少。

### URL编解码

- 网络标准RFC 1738做了硬性规定 :只有字母和数字[0-9a-zA-Z]、一些特殊符号“$-_.+!*'(),”[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL;

### Big Endian和Little Endian

> 字节序，也就是字节的顺序，指的是多字节的数据在内存中的存放顺序。

- 大端
    - Big Endian 是指低地址端 存放 高位字节。 
- 小端
    - Little Endian 是指低地址端 存放 低位字节。
- 应用
    - Java采用Big Endian来存储数据、C\C++采用Little Endian。在网络传输一般采用的网络字节序是BIG-ENDIAN。和Java是一致的。

### 如何解决乱码问题

## 语法糖

### Java中语法糖原理、解语法糖

- 语法糖（Syntactic Sugar），也称糖衣语法，是由英国计算机学家 Peter.J.Landin 发明的一个术语，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。
- 解语法糖
    - Java虚拟机并不支持这些语法糖。这些语法糖在编译阶段就会被还原成简单的基础语法结构，这个过程就是解语法糖。

### 常见语法糖原理：

#### switch 支持 String 与枚举

- Java中的`switch`自身原本就支持基本类型。
    - 比如`int`、`char`等。对于`int`类型，直接进行数值的比较。对于`char`类型则是比较其ascii码。所以，对于编译器来说，
    - `switch`中其实只能使用整型，任何类型的比较都要转换成整型。比如`byte`。`short`，`char`(ackii码是整型)以及`int`。
- String 的Switch
    - 字符串的switch是通过`equals()`和`hashCode()`方法来实现的。
- 泛型
    - `Code specialization`和`Code sharing`。
        - C++和C#是使用`Code specialization`的处理机制
        - Java使用的是`Code sharing`的机制。
    - Code sharing方式为每个泛型类型创建唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类形实例映射到唯一的字节码表示是通过类型擦除（`type erasue`）实现的。
    - Java虚拟机：**需要在编译阶段通过类型擦除的方式进行解语法糖**
        - 类型擦除主要过程：
            -  1.将所有的泛型参数用其最左边界（最顶级的父类型）类型替换。
            -  2.移除所有的类型参数。

#### 自动装箱与拆箱

- 自动装箱就是Java自动将原始类型值转换成对应的对象
- **装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。**

#### 方法变长参数

- 可变参数(`variable arguments`)是在Java 1.5中引入的一个特性。它允许一个方法把任意数量的值作为参数。

#### 枚举

- Java SE5提供了一种新的类型-Java的枚举类型，关键字`enum`可以将一组具名的值的有限集合创建为一种新的类型，而这些具名的值可以作为常规的程序组件使用，这是一种非常有用的功能。

#### 内部类

- 内部类之所以也是语法糖，是因为它仅仅是一个编译时的概念，`outer.java`里面定义了一个内部类`inner`，一旦编译成功，就会生成两个完全不同的`.class`文件了，分别是`outer.class`和`outer$inner.class`。所以内部类的名字完全可以和它的外部类名字相同。

#### 条件编译

> Java语法的条件编译，是通过判断条件为常量的if语句实现的。其原理也是Java语言的语法糖。根据if判断条件的真假，编译器直接把分支为false的代码块消除。通过该方式实现的条件编译，必须在方法体内实现，而无法在正整个Java类的结构或者类的属性上进行条件编译，这与C/C++的条件编译相比，确实更有局限性。在Java语言设计之初并没有引入条件编译的功能，虽有局限，但是总比没有更强。

- 让编译器只对满足条件的代码进行编译，将不满足条件的代码舍弃，这就是条件编译。

####  断言

- assert`关键字是从JAVA SE 1.4 引入的，为了避免和老版本的Java代码中使用了`assert`关键字导致错误，Java在执行的时候默认是不启动断言检查的（这个时候，所有的断言语句都将忽略！），如果要开启断言检查，则需要用开关`-enableassertions`或`-ea`来开启。
- 其实断言的底层实现就是if语言，如果断言结果为true，则什么都不做，程序继续执行，如果断言结果为false，则程序抛出AssertError来打断程序的执行。

#### 数值字面量

- 在java 7中，数值字面量，不管是整数还是浮点数，都允许在数字之间插入任意多个下划线。
- 编译器并不认识在数字字面量中的`_`，需要在编译阶段把他去掉

#### for-each

- for-each的实现原理其实就是使用了普通的for循环和迭代器。

#### try-with-resource

- 那些我们没有做的关闭资源的操作，编译器都帮我们做了。

#### Lambda表达式

- 实现方式其实是依赖了几个JVM底层提供的lambda相关api
- lambda表达式的实现其实是依赖了一些底层的api，在编译阶段，编译器会把lambda表达式进行解糖，转换成调用内部api的方式。

本地变量类型推断、record

# JMS

什么是Java消息服务

JMS消息传送模型

# JMX

java.lang.management.*

javax.management.*

# Java 8

lambda表达式

Stream API

时间API