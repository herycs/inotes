# RCMySQL事务和锁

## 事务特行

ACID

> 原子性
>
> - 日志
>
> 一致
>
> 隔离
>
> 持久

## 原子性

​	重点

### undo

为了实现事务一致性

mysql的nnodb中用Undo Log多版本并发控制（MVCC）

#### 修改数据流程：

![](D:\Wp\wpMD\images\imgTemp\mysqldata.png)

进程内存(Buffer)---》OS内存（OSBuffer）----》磁盘(Logfile)

方式：

DML操作

> 1.
>
> commit--->logBuffer--->OS Buffer--->file
>
> 2.默认方式
>
> commit--->file
>
> 3.
>
> commit--->OSBuffer--->file

以上方式2最安全

- redo

- bin（MysqlServer层级，任何存储引擎都有）

其他

- relay

- slow

## 事务隔离级别

读未提交read uncommit

读已提交read commit

可重复读repeatable read

串行化serializable

## 读取

数据库表中有两列用户不可见

createtime，deletetime{其中存放的是当前事务的版本号}

查询一个值，事务中实质上是查询

> createtime <= 当前事务ID
>
> deletetime > 当前事务ID || deletetime == null

RC 读取最新快照

RR 依据数据历史版本，在开启事务之前读取数据（实现采用锁）

## 幻想数据-间隙锁

### 间隙锁

区间：

(-oo, 1) 1行锁 (2,3) 3行锁 (4,6) 6行锁 (6, +oo)

### 临键锁（next key)

(-oo, 1] (1, 3) [3, +oo]

## Innodb加锁

本质是锁索引

这种特征表明：

- 只有通过索引条件检索数据，InnoDB才使用**行**级锁

- 否则，InnoDB 将使用**表**级锁

## 写锁

InnoDB：排它锁

MySAM：独占锁

## 自增锁

当前10条数据，insert 10条，但失败了

下一次insert时，id = 21

## 意向锁

加锁时判断上一级是否有锁

加行级锁，判断是否有表级锁

## 一致性锁定读

```sql
- lock in share mode 
- for update
```

**默认一致性非锁定读**

## 锁

> 共享锁
>
> 排它锁
>
> 独占锁
>
> 临键锁
>
> 间隙锁
>
> 自增锁
>
> 意向锁

wal日志

write ahead log

## 故障及故障恢复

