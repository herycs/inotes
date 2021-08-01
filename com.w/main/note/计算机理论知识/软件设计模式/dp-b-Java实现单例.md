# Java实现单例

## 目的

提供全局唯一的对象

## 问题

1. 构建实例的方法不暴露（避免外部构建新实例化）
2. 加载时机的考虑
3. 提供获取实例的方法（因为无法创建实例，故方法设置为static）
4. 线程安全
5. 序列化与反序列化的影响

### 序列化问题

如下类就是我们的单例类，设置其可以序列化，但拒绝提供读取方法，（默认序列化ID）

 默认情况下，也就是不声明serialVersionUID 属性情况下，系统会按当前类的成员变量计算hash值并赋值给serialVersionUID

检测：

```java
public class SingletonTest {
    public static void main(String[] args) throws Exception {
    Class<?> aClass = Class.forName("com.w.designPattern.create.singleton.DoubleCheck1");
    DoubleCheck1 d = (DoubleCheck1) aClass.getMethod("getInstance").invoke(Object.class);
    DoubleCheck1 a = (DoubleCheck1) aClass.getMethod("getInstance").invoke(Object.class);

    System.out.println(d == a);

    System.out.println("序列化 == 反序列化？" + (check() ? "yes" : "No"));

    while (true) {}
}
// 校验序列化结果
public static boolean check() throws IOException, ClassNotFoundException {
    doSerializable();
    DoubleCheck1 d1 = reSerializable(); // 反序列化得到对象1
    DoubleCheck1 d2 = reSerializable(); // 反序列化得到对象2
    System.out.println("d1.toString = " + d1.toString());
    System.out.println("d2.toString = " + d2.toString());
    return d1 == d2;
}
    // 序列化
    public static void doSerializable() throws IOException {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("ObjSerializable.txt"));
        objectOutputStream.writeObject(DoubleCheck1.getInstance());
        objectOutputStream.close();
    }
    // 反序列化
    public static DoubleCheck1 reSerializable() throws IOException, ClassNotFoundException {
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("ObjSerializable.txt"));
        DoubleCheck1 resObj =  (DoubleCheck1) objectInputStream.readObject();
        objectInputStream.close();
        return resObj;
    }
}
```

若单例类，不复写读取对象方法，则多次反序列化结果并不相同

```java
public class DoubleCheck1 implements Serializable {
    // private static final long serializableID = 4928832861296252415L;
    private volatile static DoubleCheck1 singleton;

    private DoubleCheck1() {}

    public static DoubleCheck1 getInstance() {
        if (singleton == null) {
            synchronized (DoubleCheck1.class) {
                if (singleton == null) {
                    singleton = new DoubleCheck1();
                }
            }
        }
        return singleton;
    }
}
/*
	true
    d1.toString = com.w.designPattern.create.singleton.DoubleCheck1@2d98a335
    d2.toString = com.w.designPattern.create.singleton.DoubleCheck1@16b98e56
    序列化 == 反序列化？No
*/
```

提供读取对象方法，则获取单例类是同一个对象

```java
public class DoubleCheck1 implements Serializable {
    // private static final long serializableID = 4928832861296252415L;
    private volatile static DoubleCheck1 singleton;

    private DoubleCheck1() {}

    public static DoubleCheck1 getInstance() {
        if (singleton == null) {
            synchronized (DoubleCheck1.class) {
                if (singleton == null) {
                    singleton = new DoubleCheck1();
                }
            }
        }
        return singleton;
    }
    public Object readResolve() {
        return singleton;
    }
}
/*
    true
    d1.toString = com.w.designPattern.create.singleton.DoubleCheck1@45ee12a7
    d2.toString = com.w.designPattern.create.singleton.DoubleCheck1@45ee12a7
    序列化 == 反序列化？yes
*/
```

## 实现

### 懒汉

线程不安全

```java
class LazyModel1 {
    private static Singleton singleton;
    private LazyModel1(){}
    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

线程安全

```java
class LazyModel2 {
    private static Singleton singleton;
    private LazyModel2(){}
    //lazy loading v1.1---线程安全，但冗余
    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

### 饿汉---OK

线程安全

```java
class HungerModel {
    private static Singleton singleton = new Singleton();
    private HungerModel() {}
    public static Singleton getSingleton() {
        return singleton;
    }
}
```

### 双重校验锁---特殊需求

DCL double check locking

#### 线程安全

```java
// lazy loading v1.3---双重检查锁,线程安全 volatile --- 序列化问题
class LazyModel4 {
    private static volatile Singleton singleton;
    private LazyModel4(){}
    //双重检查锁
    public static Singleton getInstance(){
        if (singleton == null) {
            synchronized (LazyModel4.class){
                if (singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

#### 无序列化问题

```java
// 双重检查锁，解决序列化问题-------->YES
class DoubleCheck implements Serializable {
    private volatile static Singleton singleton;
    private DoubleCheck() {}
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (DoubleCheck.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
    public Object readResolve() {
        return singleton;
    }
}
```

### 内部类---明确lazy loading效果

```java
class InnerClass {
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    private InnerClass() {
    }
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

### 枚举---反序列化

普通的 Java 类的反序列化过程中，会通过反射创建新的实例。而枚举在序列化的时候仅是将枚举对象的 name 属性输出到结果中，反序列化的时候则是通过 java.lang.Enum 的 valueOf 方法来根据名字查找枚举对象。同时，编译器是不允许任何对这种序列化机制的定制的，禁用了writeObject、readObject、readObjectNoData、writeReplace 和 readResolve 等方法。

```java
// 枚举，无序列化问题-------->YES
enum EnumModel {
    INSTANCE;
    EnumModel() {}
}
```

## 使用

1. 饿汉
2. lazy 效果 内部类
3. 反序列化，枚举
4. 其他需求，DCL（复写读对象方法）