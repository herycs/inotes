# String

## final修饰？

> final修饰的**引用**被限定为**不可变**

1. 实现字符串常量池
2. 安全，使用字符串处理的一些重要参数
3. hashCode无需再次计算，String创建是hashCode已经计算过了，当作为map中的键时非常方便

## 字符串的不可变性

- 一个string对象在内存(堆)中被创建出来，他就无法被修改
- 可修改的字符串
    - 应该使用StringBuffer 或者 StringBuilder

## substring

### JDK 6

#### 过程

- 当调用substring方法的时候会创建一个新的string对象，但是这个string的仍然指向堆中**原字符数组**。这两个对象中只有count和offset 的值是不同的，可能导致内存泄露。

#### 原码

```java
//JDK 6
String(int offset, int count, char value[]) {
    this.value = value;
    this.offset = offset;
    this.count = count;
}

public String substring(int beginIndex, int endIndex) {
    //check boundary
    return  new String(offset + beginIndex, endIndex - beginIndex, value);
}
```

#### 问题？

字符串很大，但裁剪了部分，这回导致大部分没有用到的空间也无法回收

解决办法：在JDK 6中，一般用x.substring(x, y) + ""来解决该问题，原理其实就是生成一个新的字符串并引用他

### JDK 7 

- 在jdk 7 中，substring方法会在堆内存中创建一个**新的数组**
- 1.7解决本质：使用`new String`创建了一个新字符串，避免对老字符串的引用。

### String对“+”的重载

- String s = "a" + "b"，编译器会进行常量折叠(因为两个都是编译期常量，编译期可知)，即变成 String s = "ab"
- 对于能够进行优化的(String s = "a" + 变量 等)用 StringBuilder 的 append() 方法替代，最后调用 toString() 方法 (底层就是一个 new String())

## 拼接的方式和区别

### +拼接

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

### concat

- 创建了一个字符数组，长度是已有字符串和待拼接字符串的长度之和，再把两个字符串的值复制到新的字符数组中，并使用这个字符数组创建一个新的String对象并返回

### StringBuffer

未固定长度，append(）方法修改

```java
//源码
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```

### StringBuilder

未固定长度，append(）方法修改

```java
//源码
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

### 扩展包StringUtils.join

StringUtils.join更擅长处理字符串数组或者列表的拼接。

+拼接字符串

- `for`循环中，每次都是`new`了一个`StringBuilder`，然后再把`String`转成`StringBuilder`，再进行`append`。

### 效率

```java
StringBuilder < StringBuffer < concat < + < StringUtils.join
```

### 结论

- 不是在循环体中进行字符串拼接的话，直接使用`+`
- 循环体内，字符串的连接方式，使用 `StringBuilder` 的 `append` 方法进行扩展。而不要使用`+`。
- 并发环境用StringBuffer

## String.valueOf&Integer.toString

String.valueOf(i)通过调用Integer.toString(i)

- String i1 = "" + i;
- 反编译结果:String i1 = (new StringBuilder()).append(i).toString();
- 首先创建一个StringBuilder对象，然后再调用append方法，再调用toString方法。

## switch对String的支持

> **switch中只能使用整型**

- jdk 1.7 switch支持这样几种数据类型：`byte` `short` `int` `char` `String`
- switch
    - switch对int的判断是直接比较整数的值
    - 对char类型进行比较的时候，实际上比较的是ascii码
    - 字符串的switch是通过`equals()`和`hashCode()`方法来实现的

## 字符串池

在JVM中，为了减少相同的字符串的重复创建，为了达到节省内存的目的。会单独开辟一块内存，用于保存字符串常量，这个内存区域被叫做字符串常量池。

当代码中出现双引号形式（字面量）创建字符串对象时，JVM 会先对这个字符串进行检查，如果字符串常量池中存在相同内容的字符串对象的引用，则将这个引用返回；否则，创建新的字符串对象，然后将这个引用放入字符串常量池，并返回该引用。

这种机制，就是字符串驻留或池化。

**位置**

在JDK 7以前的版本中，字符串常量池是放在永久代中的。

因为按照计划，JDK会在后续的版本中通过元空间来代替永久代，所以首先在JDK 7中，将字符串常量池先从永久代中移出，暂时放到了堆内存中。

在JDK 8中，彻底移除了永久代，使用元空间替代了永久代，于是字符串常量池再次从堆内存移动到永久代中

## 常量池

### Class常量池

（运行时常量池、Class常量池）

#### 内容

常量池中主要存放两大类常量：字面量（literal）和符号引用（symbolic references）。

#### 符号引用

-  类和接口的全限定名

-  字段的名称和描述符
-  方法的名称和描述符

#### 常量池作用

Java代码在进行`Javac`编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class文件的时候进行动态连接。也就是说，在Class文件中不会保存各个方法、字段的最终内存布局信息，因此这些字段、方法的符号引用不经过运行期转换的话无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中。

### 运行时常量池

运行时常量池（ Runtime Constant Pool）是每一个类或接口的常量池（ Constant_Pool）的运行时表示形式。

它包括了若干种不同的常量：从编译期可知的数值字面量到必须运行期解析后才能获得的方法或字段引用。运行时常量池扮演了类似传统语言中符号表（ SymbolTable）的角色，不过它存储数据范围比通常意义上的符号表要更为广泛。

每一个运行时常量池都分配在 Java 虚拟机的方法区之中，在类和接口被加载到虚拟机后，对应的运行时常量池就被创建出来。

#### 常量来源

运行时常量池中包含了若干种不同的常量：

编译期可知的字面量和符号引用（来自Class常量池） 运行期解析后可获得的常量（如String的intern方法）

所以，运行时常量池中的内容包含：Class常量池中的常量、字符串常量池中的内容

### 常量池区别

虚拟机启动过程中，会将各个Class文件中的常量池载入到运行时常量池中。

所以， Class常量池只是一个媒介场所。在JVM真的运行时，需要把常量池中的常量加载到内存中，进入到运行时常量池。

字符串常量池可以理解为运行时常量池分出来的部分。加载时，对于class的静态常量池，如果是字符串会被装到字符串常量池中。

### intern

运行期将字符串内容放置到字符串常量池的办法

## 长度限制

### 编译期

这个值只是在运行期，我们构造String的时候可以支持的一个最大长度，而实际上，在编译期，定义字符串的时候也是有长度限制的。

源码中是int 类型的长度，是可以支持2147483647(2^31 - 1)，但实际编译并非如此

字面量在编译之后会以常量的形式进入到Class常量池，要遵守**常量池**的有关规定。

《Java虚拟机规范》中CONSTANT_String_info 用于表示 java.lang.String 类型的常量对象

```java
CONSTANT_String_info {
    u1 tag;
    u2 string_index;
}
```

CONSTANT_Utf8_info 结构用于表示字符串常量的值

```java
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```

u2表示两个字节的无符号数, 16位无符号数可表示的最大值位2^16 - 1 = 65535。

当长度 >= 65535时会编译失败

### 运行期

最大长度，Integer.MAX_VALUE

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