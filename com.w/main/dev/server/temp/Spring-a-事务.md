# Spring-事务

## 事务

### 如何管理

Spring的事务管理其实就是基于DataSource的事务管理，也就是基于数据库的事务管理。

Spring提供的事务管理有两种：

l   编程式事务：(粒度控制在代码块，需要手动提交)

l   声明式事务：使用xml，注解。粒度控制在public方法。

### 隔离级别

在mysql中定义了4中隔离级别，读已提交 可从福读 读未提交可串行化

在spring的事务管理中，定义了5种隔离级别。

再次基础上 添加 默认隔离级别

### 事务传播行为

如果在开始当前事务之前，一个事务上下文已经存在了，此时有若干选项可以指定一个事务性方法的执行行为。

// 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。

int PROPAGATION_REQUIRED = 0;

// 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

int PROPAGATION_SUPPORTS = 1;

// 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

int PROPAGATION_MANDATORY = 2;

// 创建一个新的事务，如果当前存在事务，则把当前事务挂起。

int PROPAGATION_REQUIRES_NEW = 3;

// 以非事务方式运行，如果当前存在事务，则把当前事务挂起。

int PROPAGATION_NOT_SUPPORTED = 4;

// 以非事务方式运行，如果当前存在事务，则抛出异常。

int PROPAGATION_NEVER = 5;

// 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；

// 如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

int PROPAGATION_NESTED = 6;

- 事务超时时间

所谓::事务超时::就是指：一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

默认设置为底层事务系统的超时值，如果底层数据库事务系统没有设置超时值，那么就是none，没有超时限制。

- 事务只读属性

只读事务用于客户代码只能读但不修改数据的情形，只读事务用于特定情景下的优化。

Spring事务回滚规则

其实也就是在事务内部如果抛出了异常，Spring的事务回滚规则是怎么样的。spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。

**默认配置**：spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。

我们可以明确的配置在抛出那些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。

### @Transactional 注解

 \* transactionManager : 指定使用的事务管理器，这也是注解的默认值 
 \* propagation：事务传播行为 
 \* isolation：事务隔离级别 
 \* readOnly：是否是只读 
 \* timeout：超时时间 
 \* rollbackFor：导致事务回滚的异常类数组 
 \* rollbackForClassName：导致事务回滚的异常类名字数组 
 \* noRollbackFor：不会导致事务回滚的异常类数组 
 \* noRollbackForClassName：不会导致事务回滚的异常类名字数组

::注解作用域:: 
 作用于类以及类方法上。但是只能作用域public方法，因为是基于AOP实现的。