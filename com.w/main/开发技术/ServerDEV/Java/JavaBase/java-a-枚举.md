# 枚举

### 枚举的用法

定义常量

```java
public static enum course{
    math, java, os
}
```

switch中使用

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