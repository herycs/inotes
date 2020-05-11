# 并发事务与锁

## 事务

### 基础

特性

- 原子性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
- 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
- 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。
- 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

事务分类

- 自动提交事务
- 用户定义事务
- 分布式事务

### 管理

设置事务隔离级别

```sql
set 
	{global|session} 
transaction isolation level
	{read uncommitted|read committed|repeatable read|serializable}
```

获取当前隔离级别

```sql
select @@tx_isolation
```

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

### 事务并发

#### 问题

相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来一下问题：

**脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读” 

**不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。 

**幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读” 

#### 解决方式

上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证。

##### 事务隔离级别

数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

|                  | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| read uncommitted |  √   |     √      |  √   |
|  read committed  |      |     √      |  √   |
| repeatable read  |      |            |  √   |
|   serializable   |      |            |      |

通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况

```sql
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 18702 |
| Innodb_row_lock_time_avg      | 18702 |
| Innodb_row_lock_time_max      | 18702 |
| Innodb_row_lock_waits         | 1     |
+-------------------------------+-------+
--如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高
```

### 实践

前期准备

```sql
create database if not exists school default charset utf8mb4;
```

```sql
use school;
```

```sql
create table class2(
id int unique,
name char(21),
age smallint
);
```

```sql
insert into class3 values(1,'limin');
insert into class3 values(2,'zhangpeng');
insert into class3 values(3,'wangming');
```

```sql
-- 查询事务状态 1:up, 0:down
select @@autocommit;
-- 修改为0
set autocommit = 0;
```

读未提交

```sql
-- 设置事务隔离级别
set session transaction isolation level read uncommitted;

update class2 set age = 11;

-- 流程
A:
	update,do not commit
	select *
B:
	select *
	
-- 结果：脏读
-- A:未commit,B已读到update后的数据
```

读已提交

```sql
set session transaction isolation level read committed;

start transaction;

update class2 set age = 22;

-- 流程
A:
	update,do not commit
	select *
B:
	select *
-- 结果：无脏读，有不可重复读
-- A:未commit,B未读到未提交事务
-- A:commit,B读到已提交事务，但此时B事务还未结束，两次同样操作=>不一样的结果
```

可重复读

```sql
-- 设置隔离级别
set session transaction isolation level repeatable read;

update class2 set age = 33;

--流程
A:
	update, commit
	select *
B:
	select *
--结果：
A、B事务中数据不变,只有commit事务后，数据才是最新的

--流程
A:
	insert
B:
	insert the same
--结果
A插入数据，不提交，此时B查询时并不会有A插入的最新的数据（也就是B的查询结果表明此时可以插入），但插入同样的数据会处于等待状态
Acommit,B提示错误
if A do not commit,B wait
if A commit, B error
```

## 锁

### 基础

**目的**

- 并发下的数据一致性，数据有效性
- 数据库性能（锁冲突）

MySQL的锁机制比较简单，其最 显著的特点是不同的**存储引擎**支持不同的锁机制。

- MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）；
- InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。 

**锁分类**

- 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 

- 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。  

> 仅从锁的角度 来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；
>
> 而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统

**锁的特点**

- 粒度
- 隐式和显式锁
- 锁的类型
- 锁的钥匙
- 锁的生命周期

**锁定与解锁**

- 锁定表

    > 对一个事务使用**表锁定**机制时，会隐式的提交所有事务，开始一个事务时会解开所有表锁定，事务表中@@autocommit必须设定为0，否则会在执行后立刻释放表锁定，会造成死锁

    ```sql
    lock tables tbale_name[as alias]{read[local]|[low_priority]write}
    ```

- 解锁表

    ```sql
    unlock tables;
    ```

**锁分类**

表锁

- 读锁

- 写锁

行锁

- 排他锁 Exclusive Locks（简称 X 锁，属于行锁）
- 共享锁 Shared Locks  （简称 S 锁，属于行锁）
- 意向锁
    - 意向共享锁 Intention Shared Locks （简称 IS 锁，属于表锁）
    - 意向排它锁 Intention Exclusive Locks （简称 IX 锁，属于表锁）
- 自增锁 AUTO-INC Locks

页锁

### 共享锁

​		共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据库，但是只能读不能修改；

### 排他锁

​		排它锁不能与其他锁并存，如一个事务获取了一个数据行的排它锁，其他事务就不能再获取改行的锁，只有当前获取了排它锁的事务可以对数据进行读取和修改。

​		delete、update、insert默认是排他锁

### 意向共享锁和意向排他锁

意向共享锁：表示事务准备给数据行加入共享锁，也就是说一个数据行在加共享锁之前必须先取得该表的IS锁

意向排他锁：表示事务准备给数据行加入排它锁，也就是说一个数据行加排它锁之前必须先取得该表的IX锁。

**意向锁是InnoDB数据操作之前自动加的，不需要用户干预**

### 自增锁

针对自增列自增长的一个特殊的表级别锁

```sql
SHOW VARIABLES LIKE 'innodb_autoinc_lock_mode';
--默认值1 代表连续，事务未提交则id永久丢失
```

### 存储引擎的锁

#### MyISAM表锁

MySQL的表级锁有两种模式：

- 表共享读锁（Table Read Lock）
- 表独占写锁（Table Write Lock）

> 对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；
>
> 对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；
>
> MyISAM表的读操作与写操作之间，以及写操作之间是**串行**的

注意点：MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预

**MyISAM的并发插入问题**

MyISAM表的读和写是串行的，这是就总体而言的，在一定条件下，MyISAM也支持查询和插入操作的并发执行

通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺： 

```sql
mysql> show status like 'table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 352   |
| Table_locks_waited    | 2     |
+-----------------------+-------+
--如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
```

MyISAM写锁阻塞读

​		当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的读写操作都会等待，直到锁释放为止。

MyISAM读阻塞写

​		一个session使用lock table给表加读锁，这个session可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个session可以查询表中的记录，但更新就会出现锁等待。

#### InnoDB锁

InnoDB的行锁模式及加锁方法

**共享锁（s）**：又称读锁。允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。

- 若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。

**排他锁（x）**：又称写锁。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。

- 若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。

**加锁情况**

**自动**：

> mysql InnoDB引擎默认的修改数据语句：

- **update**,**delete**,**insert**都会自动给涉及到的数据加上**排他锁**
- **select**语句默认**不**加任何锁类型**，*

**主动**：

- 如果加排他锁可以使用**select …for update**语句
- 共享锁可以使用**select… lock in share mode**语句
- 加过排他锁的数据行在其他事务中是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过**select …from…**查询数据，因为普通查询没有任何锁机制。 

**行锁实现方式**

​		InnoDB行锁是通过给**索引**上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！**  

1、在不通过索引条件查询的时候，innodb使用的是表锁而不是行锁

```sql
create table tab_no_index(id int,name varchar(10)) engine=innodb;
insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| set autocommit=0<br />select * from tab_no_index where id = 1; | set autocommit=0<br />select * from tab_no_index where id =2 |
|      select * from tab_no_index where id = 1 for update      |                                                              |
|                                                              |     select * from tab_no_index where id = 2 for update;      |

session1只给一行加了排他锁，但是session2在请求其他行的排他锁的时候，会出现锁等待。原因是在没有索引的情况下，innodb只能使用表锁。

2、创建带索引的表进行条件查询，innodb使用的是行锁

```sql
create table tab_with_index(id int,name varchar(10)) engine=innodb;
alter table tab_with_index add index id(id);
insert into tab_with_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| set autocommit=0<br />select * from tab_with_indexwhere id = 1; | set autocommit=0<br />select * from tab_with_indexwhere id =2 |
|     select * from tab_with_indexwhere id = 1 for update      |                                                              |
|                                                              |     select * from tab_with_indexwhere id = 2 for update;     |

3、由于mysql的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现冲突的。

```sql
alter table tab_with_index drop index id;
insert into tab_with_index  values(1,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                       set autocommit=0                       |                       set autocommit=0                       |
| select * from tab_with_index where id = 1 and name='1' for update |                                                              |
|                                                              | select * from tab_with_index where id = 1 and name='4' for update<br />虽然session2访问的是和session1不同的记录，但是因为使用了相同的索引，所以需要等待锁 |

### 实践

Innodb

```sql
--间隙锁
-- (-oo, 1) 1 (1,3) 3 (3,4) 4 (4,+oo)
--临间锁
-- (-oo, 1] 1 (1,3] (3,4] (4,+oo)
A:
select * from class3 where id > 3 for update;
B:
delete 2, insert 2, success
insert 5, failed

--表锁
A:
select * from calss3 for update;
B:
insert 2, failed
```

### 死锁管理

- 一般将较小的事务进行回滚，让较大锁完成任务

### 注意事项

- 锁粒度越小，并发性能要求越高
- 事务中，避免一个事务中使用不同引擎的表
- 处理事务时选择设置和使用较低的隔离级别
- 尽量使用行锁控制的隔离级别
- InnoDB支持的行锁，设置合理的超时参数范围，编写锁等待超时异常处理程序解决锁等待问题
- 多记录修改，获取所有表的排他锁之后再执行修改操作
- 缩短锁的生命周期
- 事务中尽量按照一定顺序访问数据库对象

## 经验

**对于MyISAM的表锁** 

- 在一定条件下，MyISAM允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表查询和插入的锁争用问题。 
- MyISAM默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIORITY_UPDATES参数，或在INSERT、UPDATE、DELETE语句中指定LOW_PRIORITY选项来调节读写锁的争用。 
- 由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，MyISAM表可能会出现严重的锁等待，可以考虑采用InnoDB表来减少锁冲突。

**对于InnoDB表：** 
（1）InnoDB的行锁是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。 
（2）在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同。

减少锁冲突和死锁

- 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
- 选择合理的事务大小，小事务发生锁冲突的几率也更小；
- 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
- 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
- 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
- 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。

