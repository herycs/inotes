# 反射&代理

## 概念

运行状态下可以获取任意一个类，可以调用任意对象的方法，可以操作其属性值等

## 用处

- 运行时修改或者获取class，对象的信息
- 运行时操作对象，修改数据等等
- 解耦，运行时装配对象

## Class类

我们使用Class类来实现反射的一些操作，此类中定义了一些操作类的方法，通过此类的方法，我们可以获得类的方法属性等信息，也可调用方法或者操作属性，实际上当一个类被JVM装载如内存后就会有个java.lang.Class对象与其关联向我们提供操作这个类的方法

## 操作

基础类

```java
package com.w.JavaBase.reflect;

public class DEMO {

    private int id;
    public String name;
    volatile int age;
    transient double money;

    public DEMO() {}

    private void show(){ return; }

    private synchronized void show1(){ return; }

    public DEMO(int id) {  this.id = id; }

    public int getId() { return id; }

    public void setId(int id) {  this.id = id; }
	//...省略其他getter,setter代码
}
```

获取类

```java
DEMO demo = new DEMO();
Class aClass = demo.getClass();//实例调用方法

Class bClass = DEMO.class;//直接获取类

Class cClass = Class.forName("com.w.JavaBase.reflect.DEMO");//路径获取类
```

获取类的属性方法

- 获取类名称

    ```java
    String name = aClass.getName();//获取类名:com.w.JavaBase.reflect.DEMO
    ```

- ​	获取类字段

    ```java
    Field[] fileds = aClass.getFields();//获取public字段列表:nameclass java.lang.String
    Field[] declaredFields = aClass.getDeclaredFields();//获取所有字段列表
    /*
        private int com.w.JavaBase.reflect.DEMO.id
        public java.lang.String com.w.JavaBase.reflect.DEMO.name
        volatile int com.w.JavaBase.reflect.DEMO.age
        transient double com.w.JavaBase.reflect.DEMO.money
    */
    ```

- 获取方法

    ```java
    Method[] methods = aClass.getMethods();//获取public方法,继承方法
    /*
        public java.lang.String com.w.JavaBase.reflect.DEMO.getName()
        public int com.w.JavaBase.reflect.DEMO.getId()
        public void com.w.JavaBase.reflect.DEMO.setName(java.lang.String)
        public void com.w.JavaBase.reflect.DEMO.setId(int)
        public void com.w.JavaBase.reflect.DEMO.setAge(int)
        public double com.w.JavaBase.reflect.DEMO.getMoney()
        public int com.w.JavaBase.reflect.DEMO.getAge()
        public void com.w.JavaBase.reflect.DEMO.setMoney(double)
        （以下是Object方法，这也在一个方面证实了所有类都是Object的子类）
        public final void java.lang.Object.wait() throws java.lang.InterruptedException
        public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
        public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
        public boolean java.lang.Object.equals(java.lang.Object)
        public java.lang.String java.lang.Object.toString()
        public native int java.lang.Object.hashCode()
        public final native java.lang.Class java.lang.Object.getClass()
        public final native void java.lang.Object.notify()
        public final native void java.lang.Object.notifyAll()
    */
    Method[] declaredMethods = aClass.getDeclaredMethods();//获取类内部的所有方法
    /*
        public java.lang.String com.w.JavaBase.reflect.DEMO.getName()
        public int com.w.JavaBase.reflect.DEMO.getId()
        public void com.w.JavaBase.reflect.DEMO.setName(java.lang.String)
        public void com.w.JavaBase.reflect.DEMO.setId(int)
        public void com.w.JavaBase.reflect.DEMO.setAge(int)
        public double com.w.JavaBase.reflect.DEMO.getMoney()
        //对比上述结果可以发现此方法获取到的只有类本身的全部方法
        private synchronized void com.w.JavaBase.reflect.DEMO.show1()//是有方法
        public int com.w.JavaBase.reflect.DEMO.getAge()
        private void com.w.JavaBase.reflect.DEMO.show()//私有方法
        public void com.w.JavaBase.reflect.DEMO.setMoney(double)
    */
    Method getAge = aClass.getMethod("getAge");//获取指定方法:getAge [Ljava.lang.reflect.Parameter;@7f31245a
    ```

- 构造器获取

    ```java
    Constructor[] constructors = aClass.getConstructors();//获取所有构造器
    /*
        public com.w.JavaBase.reflect.DEMO()
        public com.w.JavaBase.reflect.DEMO(int)
    */
    ```

- 获取实例

    ```java
    Object o = aClass.newInstance();//获取类的实例
    //com.w.JavaBase.reflect.DEMO@135fbaa4
    ```

- 操作属性

    ```java
    Field age = aClass.getDeclaredField("age");
    age.set(demo, 12);//demo为实例
    ```

- 操作方法

    ```java
    Method method = aClass.getMethod("getAge");
    int ageVal  = (int) method.invoke(demo);//demo为实例
    System.out.println(ageVal);
    ```

以上我们可以确定，使用反射不仅可以获取对于类的所有（私有，非私有，继承）信息（属性，方法），还可以动态操作某一个具体的方法，属性。

## 提升效率

setAccessible()取消语言访问检查可以在一定程度上提升运行速度

## 应用

1.动态的加载类、动态的获取类的信息，构造对象以及操作类（属性、方法）

2.获取泛型信息

3.处理注解

> 推荐[阅读](https://zhuanlan.zhihu.com/p/58793884)

# 代理

## 概述

正因为我们可对类进行动态的一些操作，这也有了支持我们对类之间的解耦，基于Class类给的支持，我们可以在内存中对类进行操作，常用的动态代理就是使用反射在内存中对类实现一个动态的代理（与之相对的经常提起的也就是静态代理，这很容易理解，直白点就是代码中实现的代理操作）

## 动态代理实现

- 借助JDK的Class实现动态代理（限制：需要被代理对象实现了接口）
- 使用CGlib实现动态代理（底层借助的是ASM操作字节码）：动态生成被代理对象的子类（这种方式会导致final修饰的方法字段代理后无法操作）

## ASM

> 被用于动态生成类或者增强即有类的功能，对于ASM可以参见[网页](https://asm.ow2.io/)
>
> 相对于直接修改（文本编辑器）class文件来实现对类的改动，ASM则提供了封装好的方法，用于更加便捷的操作类（无需再耗费精力去计算便宜字段，文件校验码等）

正因为class文件的规范，其内部的数据排列之间有着严格的限制，这样在解析class文件时也比较方便，ASM框架通过封装对字节码的操作对外提供了能动态操作字节码的API（通常有两个方法：1.直接生成字节码；2.加载入JVM前操作类）

### 加载方式

1. 基于访问者模式操作类（Core）
2. 基于树节点操作类（Tree）

## Spring中的选用

- 类实现了接口，则JDK动态代理
- 类没有实现接口，则CGlib（当然可以通过设置强制使用CGlib）

# 注解

## 元注解

- 定义其他注解的注解
- 元注解有四个:
    - @Target（表示该注解可以用于什么地方）
    - @Retention（表示再什么级别保存该注解信息）
    - @Documented（将此注解包含再javadoc中）
    - @Inherited（允许子类继承父类中的注解）。

## 自定义注解

- 在Java中，类使用class定义，接口使用interface定义，注解和接口的定义差不多，增加了一个@符号，即@interface

Target值

```java
public enum ElementType {
    TYPE,//类，接口（包括注解类型）或枚举的声明
    FIELD,//属性的声明
    METHOD,//方法的声明	
    PARAMETER,//方法形式参数声明
    CONSTRUCTOR,//构造方法的声明
    LOCAL_VARIABLE,//局部变量声明
    ANNOTATION_TYPE,//注解类型声明
    PACKAGE//包的声明
}
```

## 操作

注解类定义

```java
@Target(ElementType.METHOD)//注解类型
@Retention(RetentionPolicy.RUNTIME)//注解保存级别
public @interface AnnotationDemo1{
    int id();
    String desc() default "none desc";
}
```

注解使用

```java
public class AnnotationTest {
    @AnnotationDemo1(id = 12, desc = "name")
    public int show(){
        System.out.println("方法执行了");
        return 123;
    }
}
```

利用反射获取注解值以及执行方法

```java
public static void main(String[] args) throws Exception {
    Class aClass = Class.forName("com.w.JavaBase.reflect.AnnotationTest");
    Method[] methods = aClass.getMethods();//获取类的public方法
    for(Method method : methods){
        if(method.isAnnotationPresent(AnnotationDemo1.class)){//判定是否符合注册特征
            AnnotationDemo1 anno = method.getAnnotation(AnnotationDemo1.class);//获取注解操作类
            System.out.println("id:" + anno.id() +",desc:" + anno.desc());//执行注解类方法获取注解参数值
            int result = (int) method.invoke(aClass.newInstance());//调用方法，并捕获返回值
            System.out.println("方法返回值为："+result);//打印
        }
    }
}
```

# 泛型

## 概念

允许在定义类和接⼜的时候使⽤类型参数（ type parameter）（jdk1.5引入），在设计一些通用操作类时很有作用（例如在常用的集合框架中就采用泛型进行类型的声明）

## 类型擦除

> 正因为类型可以抽象化，我们在使用的时候更加灵活

原本我们要实现一个参数化的方法可能需要Object或者其父类等，在内部使用instanceof判定后进行类型转换，但使用泛型后将类型填充后不再需要如此繁琐了

例如如下代码：

```java
public class  TypeDemo <T>{
    private T num;

    public TypeDemo() { }

    public void setNum(T num) { this.num = num; }

    public T getNum(){ return num; }
}
```

使用时只需要填充T类型即可

```java
public static void main(String[] args) {
	TypeDemo<Integer> typeDemo = new TypeDemo<>();
    typeDemo.setNum(123);
    System.out.println(typeDemo.getNum());//执行无误

    typeDemo.setNum("test");//idea会提示有错误，直接编译也会报告类型不兼容
}
```

可见使用泛型不但少去了类型转换这一步，还增加了个类型检查的功能

我们反编译操作一下TypeDemo.class，可以看到它在编译后做了类型擦除，所谓的T类型变成了Object

```java
public class TypeDemo {

    private Object num;

    public TypeDemo() { }

    public void setNum(Object obj) { num = obj; }

    public Object getNum() { return num; }
}
```

但实质上呢？

```java
//这是set()方法的字节码的反编译版本，可以看到这里面实质上保留了T类型的描述
LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/w/JavaBase/type/TypeDemo<TT;>;
            0       6     1   num   TT;
    Signature: #20                          // (TT;)V
//当将泛型位置全都替换成Integer时，可以看到TT ---> TInteger
LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/w/JavaBase/type/TypeDemo<TInteger;>;
            0       6     1   num   TInteger;
    Signature: #20                          // (TInteger;)V
```

去掉Class文件处的类型限制呢

```java
LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lcom/w/JavaBase/type/TypeDemo;
            0       6     1   num   Ljava/lang/Integer;//可以看到原来的TT或者TInteger已经消失，换来的是java中已知的类型
```

编译器处理泛型

- 1.单独编译：生成多份目标代码（类似于同一份文档生成两份中英文版，需要的时候找对应的版本即可）
- 2.共享：生成一份目标代码（类似于参数化配置文档，需要时切换其配置文件即可）

类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码上。

- 编译器只为泛型类型生成一份字节码，并将其实例关联到这份字节码上。
- 类型擦除的关键在于从泛型类型中清除类型参数的相关信息，并且再必要的时候添加类型检查和类型转换的方法。
- 类型擦除可以简单的理解为将泛型java代码转换为普通java代码，只不过编译器更直接点，将泛型java代码直接转换成普通java字节码。

类型擦除的主要过程如下：
-  1.将所有的泛型参数用其顶级类型替换（若不设置那就是Object，设置了泛型上边界那就会以上边界类为准进行类型擦除）。
-  2.移除所有的类型参数。

### 泛型带来的问题

泛型遇到重载

- 同样的操作擦除后其类型定义(参数列表)也成为了一样的，也就是这个时候出现了两个方法的签名是一致的情况

泛型遇到catch

- 自定义了一个泛型异常类GenericException，不应该尝试用多个catch取匹配不同的异常类型

泛型内包含静态变量

- 由于经过类型擦除，所有的**泛型类实例都关联到同一份字节码**上，泛型类的所有静态变量是共享的。

总结

- 1.虚拟机中没有泛型，只有普通类和普通方法,所有泛型类的类型参数在编译时都会被擦除,泛型类并没有自己独有的Class类对象。
- 2.创建泛型对象时请指明类型，让编译器尽早的做参数检查
- 3.不要忽略编译器的警告信息，那意味着潜在的`ClassCastException`等着你。
- 4.静态变量是被泛型类的所有实例所共享的。
- 5.泛型的类型参数不能用在`Java`异常处理的`catch`语句中。

### 常用泛型变量

- E - Element (在集合中使用，因为集合中存放的是元素)
- T - Type（Java 类）
- K - Key（键）

- V - Value（值）

- N - Number（数值类型）

- ？ - 表示不确定的java类型（无限制通配符类型）

- S、U、V - 2nd、3rd、4th types

#### 限定通配符和非限定通配符

- `限定通配符`对类型进⾏限制， 泛型中有两种限定通配符：

    表示类型的上界，格式为：<？ extends T>，即类型必须为T类型或者T子类 表示类型的下界，格式为：<？ super T>，即类型必须为T类型或者T的父类

- 泛型类型必须⽤限定内的类型来进⾏初始化，否则会导致编译错误。

    `⾮限定通配符`表⽰可以⽤任意泛型类型来替代，类型为

## 界限

上界：<? extends T>，定义上界，类型最大是T类型，使用时泛型类需要是当前类或者当前类的子类

下界：<? super T>，定义下界，类型最小是T类型，使用时泛型类需要是当前类或者当前类的父类

> [参考个人博客](https://hollischuang.github.io/toBeTopJavaer/#/basics/java-basic/generics)

# 异常

## Error和Exception

> 均继承自Throwable类。

- Error表⽰系统级的错误， 是java**运⾏环境内部错误**或者**硬件问题**， 不能指望程序来处理这样的问题， 除了退出运⾏外别⽆选择， 它是**Java虚拟机抛出**的。
- Exception 表⽰程序需要捕捉、 需要处理的常， 是由与**程序设计的不完善**⽽出现的问题， 程序必须处理的问题。

## 异常类型

> 受检异常（ checked exception） 
>
> ⾮受检异常（ unchecked exception）

- 我们在程序中调⽤他的时候， ⼀定要对该异常进⾏处理（ 捕获或者向上抛出） ， 否则是⽆法编译通过的。
- 对于⾮受检异常来说， ⼀般是运⾏时异常， 继承⾃RuntimeException。 在编写代码的时候， 不需要显⽰的捕获，但是如果不捕获， 在运⾏期如果发⽣异常就会中断程序的执⾏。

## 异常相关关键字

- throw语句⽤来明确地抛出⼀个异常；
- throws⽤来声明⼀个⽅法可能抛出的各种异常；

## 异常处理

1. 进行处理
2. 抛出交给调用者处理

## 自定义异常

- 继承`Exception`的⼦类

## 异常链

try-with-resources

- 关闭资源的常用方式就是在finally块里是释放

finally和return的执行顺序

- `try()` ⾥⾯有⼀个`return`语句， 那么后⾯的`finally{}`⾥⾯的code会不会被执⾏
- 如果try中有return语句， 那么finally中的代码还是会执⾏。
    - 因为return表⽰的是要整个⽅法体返回， 所以，finally中的语句会在return之前执⾏。
    - 但是return前执行的finally块内，对数据的修改效果对于引用类型和值类型会不同