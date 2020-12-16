# 类延迟加载

## 双重检查锁

- 问题

    ```java
    public class DoubleCheckedLocking {
        private DoubleCheckedLocking doubleCheckedLocking;
        //    双重检查锁（存在问题）
        private DoubleCheckedLocking() {}
        public DoubleCheckedLocking getInstance2(){
            if (doubleCheckedLocking == null){
                synchronized (DoubleCheckedLocking.class){
                    if (doubleCheckedLocking == null){
                        doubleCheckedLocking = new DoubleCheckedLocking();
                    }
                }
            }
            /**
             * 执行到此可能存在问题：
             *  1.doubleCheckedLocking可能未初始化结束
             *      类初始化
             *      {
             *          malloc()//分配空间-1
             *          init()//初始化-2
             *          changeRef()//修改指针指向分配空间-3
             *      }
             *      上述，2,3并无绝对顺序，并且2,3间的重排序并不违反Java语言规范的要求,但此时返回的实例却不是一个正确的实例
             * */
            return doubleCheckedLocking;
        }
    }
    ```

- 优化（这种方式相对于类加载的初始化操作而言操作，还可以延迟初始化**实例字段**）

    ```java
       //    双重检查锁-优化
        /**
         * 具体操作：使用 volatile 修饰变量，确保其初始化先于instance指针对地址的指向
         * */
        private volatile DoubleCheckedLocking doubleCheckedLocking2;
    
        public DoubleCheckedLocking2 getInstance3(){
            if (doubleCheckedLocking2 == null){
                synchronized (DoubleCheckedLocking.class){
                    if (doubleCheckedLocking2 == null){
                        doubleCheckedLocking2 = new DoubleCheckedLocking2();
                    }
                }
            }
            return doubleCheckedLocking2;
        }
    ```

## 初始化

> 处理本质是不阻止类初始化过程的重排序，而是在初始化结束之前并不暴露对象的引用

```java
public class ClassInit {
    //定义在：class中的静态变量会在类初始化时交由JVM操作（而这是线程安全的,初始化时会去获取类的初始化锁）
    private static class InstanceHolder{
        public static ClassInit classInit = new ClassInit();
    }
    //获取实例方法：直接调用静态内部类中的静态实例即可
    public static ClassInit getInstance(){
        return InstanceHolder.classInit;
    }
}
```
