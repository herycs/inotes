# Java实现单例

## 目的

提供全局唯一的对象

## 问题

1. 构建实例的方法不暴露（避免外部构建新实例化）
2. 加载时机的考虑
3. 提供获取实例的方法（因为无法创建实例，故方法设置为static）
4. 线程安全
5. 序列化与反序列化的影响

## 实现

### 枚举

```java
// 枚举，无序列化问题-------->YES
enum EnumModel {
    INSTANCE;
    EnumModel() {}
}
```

### 双重检查锁

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
class DoubleCheck{
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
    private Object readResolve() {
        return singleton;
    }
}
```