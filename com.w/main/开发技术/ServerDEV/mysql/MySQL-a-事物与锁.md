# 并发事务与锁

# 事务

## 特性

ACID

- 原子性

- 一致性

- 隔离性

- 持久性

## 隔离级别

read ncommited

read commited

read repeated(默认)

serializable

## 事务分类

- 扁平事务
- 带保存点的扁平事务
- 链事务
- 嵌套事务
- 分布式事务

### 扁平事务

所有操作一个级别，操作都是原子的，共进退

带保存点的扁平事务

允许回滚到任意保存点的状态，类似存在存档的恢复，系统崩溃时，保存点会丢失

commit并不影响获取到的锁

### 链事务

保存点模式的变种，思想是在提交一个事务时，释放不需要的数据对象，将必要的处理上下文交给下一个事务（提交事务操作和下一个事务操作将合并为一个原子操作）

回滚只能到当前事务保存点状态

commit后释放所有锁

### 嵌套事务

InnoDB原生不支持

顶层事务控制着各层子事务

### 分布式事务

## 实现

原子，一致，持久------redo log和undo log实现

redo恢复提交事务修改的页操作，undo回滚行记录到某个特定版本

redo通常是物理日志，undo是逻辑日志，记录行操作

### redo

ACID------D持久

组成：内存中的重做日志缓冲（redo log buffer），重做日志文件（redo log buffer）

redo 保证事务一致性，undo帮助事务进行回滚和MVCC

redo log顺序写，undo log随机写

为确保日志写入文件，每次重做日志缓冲写人缓冲后进行一次fsync，允许用户设置磁盘刷新策略

innodb_flush_log_at_trx_commit = {0，1(default)， 2}控制重做日志刷新到磁盘

- 0------事务提交时不写入重做日志操作
- 1------事务提交时触发fsync操作
- 2------事务提交时将重做日志写入日志文件，但只写入文件系统缓存

log block

从左日志文件使用log block存储，512字节

log group

无实际存储，逻辑上的概念

重做日志格式

redo_log_type | space | page_no | redo log body

从做日志类型，表空间ID，页偏移量

存储管理基于页，故日志也是基于页的

LSN

Log Squence Number 日志序列号，8字节，单调递增

- 重做日志写入量
- checkpoint位置
- 页的版本

恢复

物理日志，恢复比undo快

### undo

只是将数据库逻辑的恢复到原貌

存储管理采用段的方式

存储引擎有rooback segment，每个段记录了1024个 undo log segment，在每个段中进行undo页申请

purge

group commit

## 事务管理

关闭自动提交事务

```sql
set @@autocomit = 0;
```

启动事务

```sql
start transaction|begin work
```

结束事务

```sql
commit [work][and[no]chain] [[no] release]
```

回滚事务

```sql
rollback [work][and[no]chain] [[no] release]
```

设置事务检查点

```sql
savepoint identifier
-- identfier:检查点名称

rollback [work] to savepoint identifer
-- 事务回滚到某个保存点，该保存点后的保存点会被删除
```

## 事务并发

设置事务隔离级别

```sql
set {global|session} transaction isolation level{read uncommitted|read committed|repeatable read|serializable}
```

获取当前隔离级别

```sql
select @@tx_isolation
```

# 锁

> lock与latch
>
> - latch轻量级锁（mutex互斥锁， rwlock读写锁）
> - lock面向事务（锁数据库中的对象，表，页和行）

## 分类

### 锁

行锁

S Lock：共享，允许事务读一行数据

X Lock：排他，允许事务删除或者更新一行数据

表锁

IS Lock：意向共享锁，事务想要获取某几行的共享锁

IX Lock：意向排他锁，事务想要获取某几行的排它锁

表锁

- 读锁

- 写锁

行锁

- 排他锁
- 共享锁
- 意向锁
    - 意向共享锁
    - 意向排它锁

页锁

### 读写类型

一致性非锁定读

consistent nonlocking read，使用多版本并发控制获取当前执行时间数据库中的行的数据，若正在执行Update或Delete，会获取到一个快照数据（使用undo段实现）

不同事务隔离级别情况下READ COMMITTED，REPEATABLE READ情况下使用非锁定一致性读，但快照要求不一样

- READ COMMITTED------最新快照
- REPEATABLE READ------事务开始的行数据版本

一致性锁定读

InnoDB提供的操作

- select...for update---加X锁，其他事物不可加任何锁
- select...lock in share mode------加S锁，其他事物可以加S锁，加S锁时会被阻塞

自增长&锁

外键&锁

对于外键，若没有加索引，则会自动增加一个索引避免表锁

## 操作

- 锁定表

    > 对一个事务使用表锁定机制时，会隐式的提交所有事务，开始一个事务时会解开所有表锁定，事务表中@@autocommit必须设定为0，否则会在执行后立刻释放表锁定，会造成死锁

    ```sql
    lock tables tbale_name[as alias]{read[local]|[low_priority]write}
    ```

- 解锁表

    ```sql
    unlock tables;
    ```

## 算法

### 行锁

- Record Lock：**单记录锁**
- Gap Lock：间隙锁，不包含记录本身
- Next-Key Lock：Gap Lock+ Record Lock实现对于一个范围的加锁，并锁定记录本身

Record Lock：单记录锁总是锁索引记录，若没有索引，则使用隐式主键锁定

Next-Key Lock：锁定区间，技术称为Next-Key Locking，是谓词锁的一种改进，设计目的为了解决Phantom Problem(幻像问题)

​	若索引有3个值：3，7，9，则锁定区间是(-oo, 3], (3, 7], (7, 9], (9, +oo)

另外还有（Previous-Key Locking)

​	以上述为例，锁的区间是：(-oo, 3), [3, 7), [7, 9), [9, +oo)

## 锁问题

- 脏读
- 不可重复读
- 丢失更新
- 阻塞
- 死锁

### 脏读

> read ncommited

读到未提交数据

实质是数据库缓冲池中的脏页未及时刷新

### 不可重复读

> read commited

读了已提交的数据，Phantom Problem所谓的幻读，同一个事务下同一个操作执行两次结果却不相同

```sql
事物A：读取到本地a=22--->修改 age = 23--->提交
事物B：读取到本地a=22-------------------->查询a=23 !=事物前期读取到的值
```

Next-key Lock解决

### 丢失更新

事物并行操作，都对同一份原数据拷贝进行操作，执行完又并行提交，结果由一方的修改丢失了

```sql
事物A：读取到本地name = 'll', age = 22--->修改  age = 23  --->提交
DB------------------------a = 2--------------> 结果：name = 'll', age = 23或者name = 'mm', age = 22,有1方的修改丢失
事物B：读取到本地name = 'll', age = 22--->修改 name = 'mm'--->提交
```

解决：事物串行化执行

### 阻塞

一般由于等待锁导致

### 死锁

#### 处理

- 一般将较小的事务进行回滚，让较大锁完成任务

#### 注意事项

- 锁粒度越小，并发性能要求越高
- 事务中，避免一个事务中使用不同引擎的表
- 处理事务时选择设置和使用较低的隔离级别
- 尽量使用行锁控制的隔离级别
- InnoDB支持的行锁，设置合理的超时参数范围，编写锁等待超时异常处理程序解决锁等待问题
- 多记录修改，获取所有表的排他锁之后再执行修改操作
- 缩短锁的生命周期
- 事务中尽量按照一定顺序访问数据库对象

## 锁降级

**查询列为唯一索引**：对于主键加锁，由于其是唯一的，所以加锁Next-key Lock会降级为Record Lock，锁定行本身

**对于查询列不是唯一索引**：

```sql
a	b---------其中a为主键primary key, b为普通键
1	1
3	1
5	3
7	9
```

当查询 b = 3，索引有两个

对于唯一索引：a 使用record lock锁定记录 a = 5

对于非聚簇索引：b 使用next-key lock锁定 b = （1,3），InnoDB会对下一条索引增加gap lock，所以最终结果是：Lock （1,3]

