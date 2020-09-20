# Spring事务

> IOC和AOP的交织

## 设计

- 读取&处理IOC容器中配置事务处理属性，并转换为事务处理所需的内部数据结构
- Spring事务处理模块实现统一的事务处理过程，处理事务配置属性，线程绑定处理事务
- 底层事务处理，委托给事务处理器PlatformTransactionManager

## 实现

IOC创建初期设置相关属性等

AOP创建代理对象，负责在调用方法前触发对拦截器的调用，进而达到事务相关事件的处理

### 事务处理拦截器

使用：IOC容器中配置TransactionProxyFactoryBean，这是个FactoryBean

```java
1.IOC注入时创建TransactionInterceptor->创建TransactionAttributePointcut//为读取TransactionAttribute做准备
2.AbstractSingletonProxyFactoryBean->afterPropertiesSet()//实例化ProxyFactory，同时设置通知，目标对象，返回代理对象
3.Proxy对象调用方法时会调用相应的TransactionInterceptor拦截器，依据事务属性进行配置，从而为事务做好准备
```

