# SpringAOP

## 通知

- 前置通知
- 后置通知
- 环绕通知
- 返回通知之后
- 抛出通知

## 代理

官方文档指出：Spring AOP默认将标准JDK动态代理用于AOP代理。默认情况下，如果业务对象未实现接口，则使用CGLIB。

## 关于AspectJ

SpringAOP和AspectJ都是----------AOP思想的实现

但SpringAOP的使用语法反人类，，，于是借鉴了AspectJ的语法风格

Aspect-Oriented Programming

 Aspect是一种新的模块化机制，用来描述分散在对象，类或函数中的横切点关注。从关注点分离出横切关注点是面向切面的程序设计核心概念。

让算尽可能靠近数据

面向对象可以很好的组织代码，也可以很好的通过继承实现复用，但总是有功能重复的代码，且需要用在不同的地方

## 概览

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917231938318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMDcwMTc5,size_16,color_FFFFFF,t_70#pic_center)

## Advice通知-dowhat

定义做什么，为切面增强提供接口

目标方法调用前，调用

```java
public interface MethodBeforeAdvice extends BeforeAdvice {
    /*
    	method: 目标方法反射对象
    	object[]: 目标方法输入参数
    	
    */
	void before(Method method, Object[] args, @Nullable Object target) throws Throwable;
}
```

## AfterReturning

目标方法调用返回前，调用

```java
public interface AfterReturningAdvice extends AfterAdvice {
	void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable;

}
```

## PointCut

决定通知应该用于那个连接点

```java
public interface Pointcut {

	ClassFilter getClassFilter();

	MethodMatcher getMethodMatcher();

	Pointcut TRUE = TruePointcut.INSTANCE;

}
```

## Advisor

Advice-通知器-Pointcut

## AOP的设计与实现

### 代理对象

```java
ProxyFactoryBean//配置与调用此类完成代理对象生成，封装了主要代理对象的生成
```

继承自ProxyCofig类，其他的ProxyFactoryxx类实质上是对ProxyFactoryBean的调用的封装

### 配置ProxyFactoryBean

1. 定义Advisor
2. 定义ProxyFactoryBean
3. 定义Target

### ProxyFactoryBean生成FactoryBean

```java
FactoryBean-》getObject()//入口
    1.对通知器链进行初始化，（基于配置文件读取一系列的拦截器）
```

上述过程完成，则为代理对象生成准备好了环境

代理对象有两类，Singleton和Prototype

```java
public Object getObject() throws BeansException {
    initializeAdvisorChain();//初始化通知器链，实现内部有一个表示位advisorChainInitialized，若标志为初始化完成-》直接返回
    if (isSingleton()) {//处理单例类
        return getSingletonInstance();
    }
    else {//处理原型类
        if (this.targetName == null) {
            logger.info("Using non-singleton proxies with singleton targets is often undesirable. " +
                        "Enable prototype proxies by setting the 'targetName' property.");
        }
        return newPrototypeInstance();
    }
}
```

```java
getSingletonInstance()//获取单例对象,ProxyFactoryBean生成AopProxy对象的调用入口
```

Spring利用AopProxy对象将代理对象的实现和框架其他部分区分开，该接口有两个具体之类

```java
Cglib2AopProxy
JdkDynamicProxy
```

代理对象生成

```java
ProxyFactoryBean基类AdvisorSupport中借助AopProxyFactory生成AopProxyFactory
	DefaultAopProxyFactory-》选择具体的Aop对象生成
    	->JdkDynamicAopProxy
    	->CglibAopProxy
```

```java
// DefaultAopFactory
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (!IN_NATIVE_IMAGE &&
        (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config))) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: " +
                                         "Either an interface or a target is required for proxy creation.");
        }
        // 接口类，则使用Jdk动态代理
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        // cglib生成
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```

生成过程中如下：

1. AdvisedSupport中取得目标对象
2. 检查目标对象的配置
3. 基于配置选定生成方式

### JDK生成AopProxy

```java
JdkDynamicAopProxy
```

### Cglib生成AopProxy

Cglib需要指定ClassLoader

```java
Cglib2AopProxy
```

## 拦截器

### 设计原理

AOP生成代理对象时已将拦截器设置到到具体的代理对象中去了

### JDK拦截

```java
JdkDynamicAopProxy-》invoke
```

### Cglib拦截

```java
DynamicAdvisedInterceptor
```

### 目标对象方法调用

调用过程：调用-》拦截器（如果有）-》具体方法

```java
AopUtils-》AopUtils.invokeJoinpointUsingReflection()//实现
```

### 拦截器链的调用

```java
ReflectiveMethodInvocation-》proceed()//调用过程中对于拦截器的选定依据matches方法进行判断
```

### 拦截器的加入

这些拦截器都是维护在一个List中的拦截器类

对于拦截器类的获取有对应的拦截器链工厂

```java
DefaultAdvisorAdapterRegister
```

### AOP高级特性

运行时热替换