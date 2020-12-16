# MVCC数据合并

## 认识

 MVCC，全称Multi-Version Concurrency Control，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

## 作用

MVCC在MySQL InnoDB中的实现主要是为了提高数据库并发性能，用更好的方式去处理读-写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读

并发读写互不冲突，读不阻塞写， 写不阻塞读

## 使用

MVCC解决读写冲突 + {悲观锁 | 乐观锁} 解决写写冲突

## 实现原理

### 表结构定义

```mysql
table
+----+------------+-----------+-------------+-----------+------+
| id | columns... | DB_TRX_ID | DB_ROLL_PTR | DB_ROW_ID | flag |
+----+------------+-----------+-------------+-----------+------+
|    |            | 6 Byte    | 7 Byte      | 6 Byte    |      |
+----+------------+-----------+-------------+-----------+------+
-- 其中：
-- DB_TRX_ID，最后一个操作此记录的tx_id
-- DB_ROLL_PTR，这个记录的上一个版本
-- 隐藏ID，当未设置主键时会生成
-- flag，为了配合回滚，标记当前记录是存在还是删除
```

### 日志 

undo log : **insert undo log** | **update undo log**

- insert undo log：代表事务在`insert`新记录时产生的`undo log`, 只在事务回滚时需要，并且在事务**提交后**可以被立即丢弃
- update undo log：事务在进行`update`或`delete`时产生的`undo log`; 不仅在事务回滚时需要，在快照读时也需要；所以不能随便删除，只有在*快速读*或*事务回滚***不涉及该日志**时，对应的日志才会被`purge`线程统一清除

效果：不同事务或者相同事务的对同一记录的修改，会导致该记录的`undo log`成为一条记录版本线性表，既链表，`undo log`的链首就是最新的旧记录，链尾就是最早的旧记录

### READ VIEW

为了不影响MVCC的正常工作，purge线程自己也维护了一个read view（这个read view相当于系统中最老活跃事务的read view）

如果某个记录的deleted_bit为true，并且DB_TRX_ID相对于purge线程的read view可见，那么这条记录一定是可以被安全清除的。

## 覆盖类数据合并

> X(写锁) - 避免丢失更新

数据操作与原有数据无关的情况，例如：原：a = ttt, 操作set a = iii 或者 set a = bbb，最终结果是 iii 或者 bbb

| 时间 | 事务A                                   | 事务B                                           | 数据库    |
| ---- | --------------------------------------- | ----------------------------------------------- | --------- |
|      |                                         |                                                 | age=444   |
|      | begin                                   | begin                                           | age=444   |
|      | update user set age = 666 where id < 4; | select * from user where id < 4;                | age=444   |
|      |                                         | update user set age = 555 where id < 4;**阻塞** | age=444   |
|      | commit;                                 |                                                 | age = 666 |
|      |                                         | 执行完成                                        | age = 666 |
|      |                                         | commit;                                         | age = 555 |

### 原始数据

```mysql
mysql> select * from user;
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           |  444 |        |
|  2 | ll           |   33 |        |
|  3 | cc           |  999 |        |
|  4 | ss           |  444 |        |
```

### 事务A

```mysql
mysql> 
Query OK, 3 rows affected (5.67 sec)
Rows matched: 3  Changed: 3  Warnings: 0

mysql> select * from user;
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           |  666 |        |
|  2 | ll           |  666 |        |
|  3 | cc           |  666 |        |
|  4 | ss           |  444 |        |

commit;
```

### 事务B

```mysql
mysql> select * from user where id < 4;
+----+------+------+------+
| id | name | age  | addr |
+----+------+------+------+
|  1 | ww   |  444 |      |
|  2 | ll   |   33 |      |
|  3 | cc   |  999 |      |

mysql> update user set age = 555 where id < 4;
-- 阻塞ing

mysql> select * from user where id < 4;
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           |  555 |        |
|  2 | ll           |  555 |        |
|  3 | cc           |  555 |        |
```

总结：数据库中使用MVCC解决了不可重复读，对当前数据区生成快照，操作期间对

## 生长类数据合并

> X（写锁） - 确保了数据修改的一致性

数据操作与原数据有关的情况，例如：原 a = 123, 操作 a  = a + 1 或者 a = a + 2, 最终结果应当是a = 126， 而不是124 或者 125

### 原始数据

```mysql
mysql> select * from user ;
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           |  543 |        |
|  2 | ll           |  543 |        |
|  3 | cc           |  543 |        |
|  4 | ss           |  432 |        |
|  5 | ll           |   17 |        |
|  6 | ll           |   19 |        |
|  7 | ll           |   20 |        |
```

### 事务A

```mysql
-- 事务开启，数据查询为原始数据
mysql> update user set age = age - 99;
mysql> select * from user;
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           |  444 |        |
|  2 | ll           |  444 |        |
|  3 | cc           |  444 |        |
|  4 | ss           |  333 |        |
|  5 | ll           |  -82 |        |
|  6 | ll           |  -80 |        |
|  7 | ll           |  -79 |        |
```

### 事务B

```mysql
-- 事务开启，数据查询显示为原始数据
mysql> update user set age = age + 1000;
mysql> select * from user; -- 阻塞，当A事务提交后写锁解除，修改量为A事务修改后的数据值
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           | 1444 |        |
|  2 | ll           | 1444 |        |
|  3 | cc           | 1444 |        |
|  4 | ss           | 1333 |        |
|  5 | ll           |  918 |        |
```

总结：数据库中使用写锁避免了并发写情况下的数据操作异常

## 插入类数据合并

> 发生阻塞，next-key算法，导致间隙被锁定

```mysql
mysql> select * from user;
+----+--------------+------+--------+
| id | name         | age  | addr   |
+----+--------------+------+--------+
|  1 | ww           | 1444 |        |
|  2 | ll           | 1444 |        |
|  3 | cc           | 1444 |        |
|  4 | ss           | 1333 |        |
|  5 | ll           |  918 |        |
|  6 | ll           |  920 |        |
|  7 | ll           |  921 |        |
|  8 | tt           | 1111 | xian   |
| 12 | tw           | 1333 | xian   |
```

### 事务A

```mysql
-- 事务A
mysql> update user set age = 111 where addr = 'xian';
Query OK, 12 rows affected (0.01 sec)
```

### 事务B

```mysql
-- 事务B
mysql> insert into user(id, name,  age, addr) values(9,'i1', 232, 'beijin'); -- 阻塞，加锁（7.8]
```

