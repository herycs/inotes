# 并发事务与锁

## 特性

ACID

- 原子性
- 一致性
- 隔离性
- 持久性

## 出现的问题

脏读：读到其他事务rollback的数据

幻读：查询出之前不存在的数据

不可重复读：同一条数据和之前的显示不一致

## 隔离级别

### read uncommited - 脏读

> 浏览访问

### read commited - 不可重复读

> 游标稳定

数据库角度讲，RC违反了RCID中的I属性

<u>除了唯一性的约束检查和外键的约束检查，存储引擎并不适用gap lock算法</u>

- 另外在v5.1中，只能工作在二进制日志为row格式下，若是在statement下则会出现问题（statement先加后删）

    这种模式下，日志的记录格式是行记录，并不是SQL语句，从而保证了正常的执行

- 将innodb_locks_unsafe_for_binlog = 1可在二进制为statement下使用read committed

### read repeated(默认) - 幻读（next-key解决）

MVCC解决不可重复读：同一事物只能读取到当前事务版本的数据

next-key lock -> 解决

### serializable - 与RU相比，性能可能更优

此级别下每个select语句后会增加 lock in share mode

主要用于分布式事务

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
