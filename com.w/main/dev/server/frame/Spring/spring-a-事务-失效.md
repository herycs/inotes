# 事务

## 基本属性

1）事物的传播行为：propagation，即当前事物方法被另一个事物方法调用时如何使用事物，默认取值为REQUIRED，即使用调用的方法的事物。

2）事物的隔离级别：isolation:指定事务的隔离级别，最常用的取值为：READ_COMMITTED，读以提交。

3）事务回滚：

  （1）noRollbackFor、noRollbackForClassName：对这个异常不进行回滚 通常情况下取默认值即可。

  （2）rollbackFor、rollbackForClassName：对这个异常进行回滚 通常情况下取默认值即可。

4）只读事务：readOnly，只能读取数据，可以帮助数据库引擎优化。如果是只读，应设置：readOnly=true

5）强制回滚时间：timeout:指定强制回滚的时间，单位秒，如该方法执行了5秒，而该属性设置的是2秒，如果到2秒了，该方法还没有执行完，该事务也会对该方法进行强制回滚。

```java
/**
     * 使用propagation 指定事物的传播行为，即当前事物方法被另一个事物方法调用时如何使用事物，
     		默认取值为REQUIRED，即使用调用方法的事物
     * REQUIRES_NEW：使用自己的事物，调用的事物方法的事物被挂起。
     * isolation:指定事务的隔离级别，    最常用的取值为：READ_COMMITTED，读以提交
     * noRollbackFor:对这个异常不进行回滚 通常情况下取默认值即可。
     * rollbackFor:对这个异常进行回滚
     * readOnly:只读事务，只能读取数据，可以帮助数据库引擎优化。如果是只读，应设置：readOnly=true
     * timeout:指定强制回滚的时间，单位秒，
     		如该方法执行了5秒，而该属性设置的是2秒，如果到2秒了，该方法还没有执行完，该事务也会对该方法进行强制回滚。
     */
@Transactional(propagation=Propagation.REQUIRED,isolation=Isolation.READ_COMMITTED,
               noRollbackFor={UserException.class},noRollbackForClassName="UserException",
               rollbackFor={UserException.class},rollbackForClassName="UserException",
               readOnly=false,timeout=100)
public void updatUser(){
    System.out.println("需要用到事务的方法");
}
```

## 事务失效情况

1.数据库是否支持事务

2.注解所在的类是否被加载为Bean

3.注解所在的方法是否别public修饰

4.是否发生了自调用问题

5.所用数据源是否支持事务管理器

6.@Transactional的扩展配置propagation是否正确

## 细节

1.数据库

数据库MySQL5.5.5之前默认MyISAM，InnoDB支持事务

2.不被spring管理

若不生成为Bean则不会被事务所管理

3.方法不是public

> 官网信息：
>
> When using proxies, you should apply the @Transactional annotation only to methods with public visibility. 
>
> If you do annotate protected, private or package-visible methods with the @Transactional annotation, no error is raised, but the annotated method does not exhibit the configured transactional settings. Consider the use of AspectJ (see below) if you need to annotate non-public methods.

4.内部调用

方法A未被注解，方法B被事务注解，A调用B是否会触发事务呢？

方法A被注解，方法B被注解，A调用B是否会触发事务呢？

当A在类内部调用B时是否会触发事务呢？以上情况，都不会

解决办法：

- 方法分离：Service拆分，从外部调用方法才会触发事务，所有将方法拆分到不同的类里面去，<u>回滚操作不方便</u>
- 注入自己：将自身类注入自己，当调用方法时通过这个注入对象调用从而达到外部调用的情景

> [参考](https://mp.weixin.qq.com/s/1TEBnmWynN4nwc6Q-oZfvw)

5.数据源未配置事务管理器

数据源若不支持事务管理器也无法完成事务

6.不支持事务

**Propagation.NOT_SUPPORTED：** 表示不以事务运行，当前若存在事务则挂起

主动不支持

7.异常被被捕获

异常触发是回滚条件，若异常被吃掉，那么事务也不会回滚

8.异常类型错误

默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如：

```
@Transactional(rollbackFor = Exception.class)
```

这个配置仅限于 `Throwable` 异常类及其子类。

## 注解

@Transactional 注解的属性

| 属性                   | 类型                                        | 描述                                                         |
| ---------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| propagation            | 枚举型：Propagation                         | 可选的传播性设置                                             |
| isolation              | 枚举型：Isolation                           | 可选的隔离性级别（默认值：ISOLATION_DEFAULT）                |
| readOnly               | 布尔型                                      | 读写型事务 vs. 只读型事务                                    |
| timeout                | int型（以秒为单位）                         | 事务超时                                                     |
| rollbackFor            | 一组 Class 类的实例，必须是Throwable 的子类 | 一组异常类，遇到时 必须 进行回滚。默认情况下checked exceptions不进行回滚，仅unchecked exceptions（即RuntimeException的子类）才进行事务回滚。 |
| rollbackForClassname   | 一组 Class 类的名字，必须是Throwable的子类  | 一组异常类名，遇到时 必须 进行回滚                           |
| noRollbackFor          | 一组 Class 类的实例，必须是Throwable 的子类 | 一组异常类，遇到时 必须不 回滚。                             |
| noRollbackForClassname | 一组 Class 类的名字，必须是Throwable 的子类 | 一组异常类，遇到时 必须不 回滚                               |

## 总结

事务失效场景：自身调用 | 异常被捕获 | 异常不识别

> [异常失效](https://www.cnblogs.com/javastack/p/12160464.html)

