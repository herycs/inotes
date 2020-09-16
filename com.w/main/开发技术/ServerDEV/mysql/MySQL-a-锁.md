# 锁

## 简介

### 为什么要有锁？

为了在并发环境下提供给用户一致的数据读取和修改

### 行级锁开销？

行级锁本身不会增加开销，只有实现本身才会增加开销，而且，InnoDB一个锁和多个锁的开销是相同的。

### MyISAM & InnoDB

InnoDB提供了表锁和行锁

MyISAM提供了表锁，若插入操作在表底部，其也可以提供一定的并发性能

### InnoDB & SQL Server

SQL Server 提供了锁，但存在锁升级，这种情况下开销会随着锁的使用增加而增加

InnoDB 提供一致性非锁定读，行级锁（行锁不会升级，故没有额外的开销）

### lock与latch

latch轻量级锁（mutex互斥锁， rwlock读写锁）

要求锁定时间非常短，目的是为了保证并发情况下的正确性，但未提供死锁检测机制

```mysql
-- 查看命令, debug版本下系信息更多
 show engine innodb mutex;
```

lock面向事务（锁数据库中的对象，表，页和行）

一般仅在事务commit | roolback后释放，具有死锁机制

## 分类

### 行级锁

- 共享锁S lock：允许事务读一行数据
- 排它锁X lock：允许事务删除或更新一行数据

InnoDB支持多粒度锁定，这种锁定允许事务在行级上的锁和表级上的锁同时存在，为了支持这种多粒度加锁，MySQL提供了意向锁(Intention Lock)

- 意向锁 Intention Lock


意向锁分为多个层级，<u>意向锁意味着事务希望在更细粒度加锁</u>

若以树型结构理解树，那么对于一行记录加(X)锁，分别需要加锁：数据库，表，页（IX），对行记录上行（X）

- 意向共享锁：IS Lock，事务想要获取某几行的共享锁
- 意向排它锁：IX Lock，事务想要获取某几行的排它锁

InnoDB1.0之前，用户只能通过命令进行锁状态查看

InnoDB1.0之后，INFORMATION_SCHEMA架构添加，INNODB_TRX，INNODB_LOCKS，INNODB_LOCK_WAITS三张表

### 页锁

### 一致性非锁定读

在隔离级别：RC & RR下会使用一致性非锁定读

基于快照进行操作，对于快照数据，因为没有事务需要对历史数据进行加锁

consistent nonlocking read，使用多版本并发控制获取当前执行时间数据库中的行的数据，若正在执行Update或Delete，并不会等待这个锁接触，而是会获取到一个快照数据（使用undo段实现）

不同事务隔离级别情况下READ COMMITTED，REPEATABLE READ情况下使用非锁定一致性读，但快照要求不一样

- READ COMMITTED------最新快照
- REPEATABLE READ------事务开始的行数据版本

### 一致性锁定读

InnoDB提供的操作

- select...for update---加X锁，其他事物不可加任何锁

- select...lock in share mode------加S锁，其他事物可以加S锁，加X锁时会被阻塞

- 锁定表

    > 对一个事务使用表锁定机制时，会隐式的提交所有事务，开始一个事务时会解开所有表锁定，事务表中@@autocommit必须设定为0，否则会在执行后立刻释放表锁定，会造成死锁

    ```sql
    lock tables tbale_name[as alias]{read[local]|[low_priority]write}
    ```

- 解锁表

    ```sql
    unlock tables;
    ```

### 自增长&锁

```mysql
select Max from t for update;
```

原理：对于含有自增长的计数器的表进行插入操作，这个计数器会被初始化，插入操作依据这个自增长的计数器值+1

这个机制是AUTO-INC Locking

为了提高性能，这个锁的释放不是在事务结束后，而是在自增操作完毕后就释放

上述操作存在隐患，单个操作还可，多个操作尤其是大量操作时<u>性能</u>消耗很大

v5.1.22开始，提供一种轻量级互斥量的自增长实现机制

提供参数<u>innodb_autoinc_lock_mode</u>控制增长

**自增长分类**：

- insert-like：所有插入语句
- simple inserts：插入前确定行数
- bulk inserts：插入前不确定行数
- mixed-mode inserts：一部分值自增长，一部分值确定

*InnoDB中自增长列必须是索引，且是第一个索引，否则出错*

MyISAM则无这种限制

### 外键&锁

自动创建索引：对于外键，若没有加索引，则会自动增加一个索引避免表锁

修改流程：对于外键值的插入和更新，首先查询父表中的记录，及select父表

采用一致性非锁定读会发生数据不一致问题，使用select...lock in share mode

## 算法

- Record Lock：**单记录锁**
- Gap Lock：间隙锁，不包含记录本身
- Next-Key Lock：Gap Lock+ Record Lock实现对于一个范围的加锁，并锁定记录本身

### Record Lock

单记录锁总是锁索引记录，若没有索引，则使用隐式主键锁定

### Next-Key Lock

> Gap + Record

锁定区间，技术称为Next-Key Locking，是谓词锁的一种改进，设计目的为了解决Phantom Problem(幻像问题)

若索引有3个值：3，7，9，则锁定区间是(-oo, 3], (3, 7], (7, 9], (9, +oo)

对于唯一键的锁定，next-key会降级record key

示例：

```mysql
-- 数据库表如下
mysql> select * from z;
+----+------+
| a  | b    |
+----+------+
|  1 |    1 |
|  3 |    1 |
|  5 |    3 |
|  7 |    6 |
| 10 |    8 |
+----+------+
-- 事务A
mysql> select * from z where b = 3 for update; 
-- 对索引b = 3，加锁 =》1.锁主键 a = 5的记录，2.锁辅助索引b = （1,3） （3,6）
+---+------+
| a | b    |
+---+------+
| 5 |    3 |
+---+------+
-- 事务B
mysql> insert into z(a, b) value(6, 5); -- 结果会阻塞住，以为非聚簇索引加锁同时也锁定了，（3,6）
mysql> insert into z(a,b) value(6,6);
Query OK, 1 row affected (0.00 sec)
```



### Previous-Key Locking

> Record + Gap

以上述为例，锁的区间是：(-oo, 3), [3, 7), [7, 9), [9, +oo)

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

## 锁状态查看

1.

```mysql
mysql> select * from information_schema.INNODB_TRX\G;
*************************** 1. row ***************************
                    trx_id: 85255
                 trx_state: LOCK WAIT
               trx_started: 2020-09-08 15:32:04
     trx_requested_lock_id: 85255:857:3:2
          trx_wait_started: 2020-09-08 15:36:20
                trx_weight: 2
       trx_mysql_thread_id: 21
                 trx_query: select * from user for update
       trx_operation_state: starting index read
         trx_tables_in_use: 1
         trx_tables_locked: 1
          trx_lock_structs: 2
     trx_lock_memory_bytes: 1136
           trx_rows_locked: 2
         trx_rows_modified: 0
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
 trx_adaptive_hash_latched: 0
 trx_adaptive_hash_timeout: 0
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
*************************** 2. row ***************************
                    trx_id: 283648230201136
                 trx_state: RUNNING
               trx_started: 2020-09-08 15:31:57
     trx_requested_lock_id: NULL
          trx_wait_started: NULL
                trx_weight: 2
       trx_mysql_thread_id: 20
                 trx_query: NULL
       trx_operation_state: NULL
         trx_tables_in_use: 0
         trx_tables_locked: 1
          trx_lock_structs: 2
     trx_lock_memory_bytes: 1136
           trx_rows_locked: 22
         trx_rows_modified: 0
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
 trx_adaptive_hash_latched: 0
 trx_adaptive_hash_timeout: 0
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
2 rows in set (0.00 sec)
```

2.lock并非十分可信，用户使用范围查找时，可能只返回第一行的主键值，若当前资源被锁定住了，又由于缓存池用量原因被刷新出缓存，这时执行结果中lock_data的值就可能是null，InnoDB并不会再从磁盘刷新一次

```mysql
mysql> select * from information_schema.INNODB_LOCKS\G;
*************************** 1. row ***************************
    lock_id: 85255:857:3:2
lock_trx_id: 85255
  lock_mode: X
  lock_type: RECORD
 lock_table: `school`.`user`
 lock_index: PRIMARY
 lock_space: 857
  lock_page: 3
   lock_rec: 2
  lock_data: 1
*************************** 2. row ***************************
    lock_id: 283648230201136:857:3:2
lock_trx_id: 283648230201136
  lock_mode: S
  lock_type: RECORD
 lock_table: `school`.`user`
 lock_index: PRIMARY
 lock_space: 857
  lock_page: 3
   lock_rec: 2
  lock_data: 1
2 rows in set, 1 warning (0.00 sec)
```

3.

```mysql
mysql> select * from information_schema.INNODB_LOCK_WAITS\G;
*************************** 1. row ***************************
requesting_trx_id: 85255
requested_lock_id: 85255:857:3:2
  blocking_trx_id: 283648230201136
 blocking_lock_id: 283648230201136:857:3:2
1 row in set, 1 warning (0.00 sec)
```

