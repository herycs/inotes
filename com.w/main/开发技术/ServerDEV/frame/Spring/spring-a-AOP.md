# AOP

Aspect-Oriented Programming

 Aspect是一种新的模块化机制，用来描述分散在对象，类或函数中的横切点关注。从关注点分离出横切关注点是面向切面的程序设计核心概念。

让算尽可能靠近数据

面向对象可以很好的组织代码，也可以很好的通过继承实现复用，但总是有功能重复的代码，且需要用在不同的地方

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

